# API Endpoints calling `getAdminFormatedAmount(MerchantStore store, BigDecimal amount)`

## Target method

```java
// sm-core/src/main/java/com/salesmanager/core/business/utils/ProductPriceUtils.java
public String getAdminFormatedAmount(MerchantStore store, BigDecimal amount) throws Exception
```

## Methodology

The repository was analyzed statically (code search + manual call-chain tracing) to find every
REST API endpoint in the `sm-shop` module whose request-handling code path, directly or
transitively, invokes `ProductPriceUtils.getAdminFormatedAmount(MerchantStore, BigDecimal)`.

Two call paths reach the target method:

1. **Direct calls** from payment and shipping integration modules:
   - `StripePayment` (`authorize`, `authorizeAndCapture`, `refund`)
   - `Stripe3Payment` (`capture`, `authorizeAndCapture`, `refund`)
   - `BeanStreamPayment` (`authorize` → `processTransaction`, `capture`, `authorizeAndCapture` → `processTransaction`, `refund`)
   - `PayPalRestPayment` (`authorize`)
   - `USPSShippingQuote` (shipping quote calculation)
2. **Indirect calls** via `PricingService.getStringAmount(BigDecimal, MerchantStore)`, which
   itself calls `getAdminFormatedAmount` (see `PricingServiceImpl.getStringAmount`):
   - `ReadableProductAttributeMapper.merge(...)` (product attribute price formatting)
   - `PayPalExpressCheckoutPayment` (`authorize`, `refund`, internal `processTransaction`)

Each candidate endpoint's controller → facade/service → integration-module chain was verified by
reading the actual method bodies (not just references), and stub/dead-code endpoints whose
implementation is commented out or short-circuited (e.g. `OrderPaymentApi.capturePayment`,
`refundPayment`, `authorizePayment`, and the `/payment/init` endpoints, whose `initTransaction()`
implementations never call the target method) were excluded.

## Result: 8 API Endpoints

| # | HTTP Method | Endpoint | Controller | Call chain to `getAdminFormatedAmount` |
|---|------------|----------|------------|------------------------------------------|
| 1 | POST | `/api/v1/auth/cart/{code}/checkout` | `OrderApi.checkout(...)` (authenticated customer) | `OrderFacade.processOrder` → `OrderService.processOrder` → `PaymentService.processPayment` → payment module `authorize`/`authorizeAndCapture` (e.g. `StripePayment`, `Stripe3Payment`, `BeanStreamPayment`, `PayPalRestPayment`, `PayPalExpressCheckoutPayment`) → `getAdminFormatedAmount` |
| 2 | POST | `/api/v1/cart/{code}/checkout` | `OrderApi.checkout(...)` (anonymous/guest customer) | Same chain as #1 |
| 3 | GET | `/api/v1/auth/cart/{code}/shipping` | `OrderShippingApi.shipping(...)` (authenticated customer) | `OrderFacade.getShippingQuote` → `ShippingService.getShippingQuote` → `ShippingQuoteModule` implementation (e.g. `USPSShippingQuote`) → `getAdminFormatedAmount` |
| 4 | POST | `/api/v1/cart/{code}/shipping` | `OrderShippingApi.shipping(...)` (anonymous customer) | Same chain as #3 |
| 5 | GET | `/api/v1/private/product/{id}/attributes` | `ProductAttributeOptionApi.attributes(...)` | `ProductOptionFacade.getAttributesList` → `ReadableProductAttributeMapper.convert` → `PricingService.getStringAmount` → `getAdminFormatedAmount` |
| 6 | GET | `/api/v1/private/product/{id}/attribute/{attributeId}` | `ProductAttributeOptionApi.getAttribute(...)` | `ProductOptionFacade.getAttribute` → `ReadableProductAttributeMapper.convert` → `PricingService.getStringAmount` → `getAdminFormatedAmount` |
| 7 | POST | `/api/v1/private/product/{id}/attribute` | `ProductAttributeOptionApi.createAttribute(...)` | `ProductOptionFacade.saveAttribute` → `ReadableProductAttributeMapper.convert` → `PricingService.getStringAmount` → `getAdminFormatedAmount` |
| 8 | PUT | `/api/v1/private/product/{id}/attribute/{attributeId}` | `ProductAttributeOptionApi.updateAttribute(...)` | `ProductOptionFacade.saveAttribute` → `ReadableProductAttributeMapper.convert` → `PricingService.getStringAmount` → `getAdminFormatedAmount` |

## Excluded endpoints (checked but do not reach the method)

| Endpoint | Controller | Reason for exclusion |
|---|---|---|
| `POST /api/v1/cart/{code}/payment/init` | `OrderPaymentApi.init(...)` | Calls `PaymentService.initTransaction` → payment module `initTransaction()`, none of whose implementations (`StripePayment`, `Stripe3Payment`, `BeanStreamPayment`, `PayPalRestPayment`, `PayPalExpressCheckoutPayment`) invoke `getAdminFormatedAmount`/`getStringAmount` (several are stubs or throw `IntegrationException("Not implemented")`) |
| `POST /api/v1/auth/cart/{code}/payment/init` | `OrderPaymentApi.init(...)` | Same as above |
| `POST /api/v1/private/orders/{id}/capture` | `OrderPaymentApi.capturePayment(...)` | Method body is entirely commented out; returns `null` |
| `POST /api/v1/private/orders/{id}/refund` | `OrderPaymentApi.refundPayment(...)` | Stub method; returns `null` |
| `POST /api/v1/private/orders/{id}/authorize` | `OrderPaymentApi.authorizePayment(...)` | Stub method; returns `null` |

## Notes

- Which specific payment/shipping integration module actually executes at runtime depends on the
  merchant store's active configuration, but at least one live implementation of each relevant
  interface (`PaymentModule`, `ShippingQuoteModule`) calls the target method, so the endpoints that
  trigger those service calls are included regardless of merchant configuration.
- CAST Imaging MCP tools were unavailable in this session (the configured server requires an API
  key prompt that wasn't supplied), so this analysis was performed via direct source code
  traversal instead.
