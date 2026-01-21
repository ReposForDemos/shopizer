# API Endpoint Documentation: `api/v1/auth/cart/{}/payment/init/` (POST)

**Application:** Shopizer-Backend  
**Endpoint:** `api/v1/auth/cart/{code}/payment/init/`  
**HTTP Method:** POST  
**Transaction ID:** 244094  
**Total Call Graph Size:** 643 nodes

---

## 1. Technical Documentation

### 1.1 Endpoint Overview

- **Full Method Name:** `com.salesmanager.shop.store.api.v1.order.OrderPaymentApi.init`
- **Method ID:** 29714
- **Source File:** `sm-shop/src/main/java/com/salesmanager/shop/store/api/v1/order/OrderPaymentApi.java`
- **Number of Code Lines:** 47
- **Cyclomatic Complexity:** 9
- **Status:** Added

### 1.2 Annotations

```java
@RequestMapping(value={"/auth/cart/{code}/payment/init"}, method=RequestMethod.POST)
@ResponseBody
@ApiImplicitParams({
    @ApiImplicitParam(name="store", dataType="String", defaultValue="DEFAULT"),
    // Additional parameters
})
```

### 1.3 Method Signature

```java
init(
    com.salesmanager.shop.model.order.transaction.PersistablePayment payment,
    java.lang.String code,
    com.salesmanager.core.model.merchant.MerchantStore store,
    com.salesmanager.core.model.reference.Language language
)
```

### 1.4 Technology Stack

The endpoint utilizes the following technology layers:
- **Hibernate** - ORM layer
- **Java** - Programming language
- **Spring** - Application framework
- **Spring Web Services** - REST API framework
- **SQL/PostgreSQL** - Database layer

### 1.5 Database Tables Accessed

The transaction interacts with the following database tables:

| Table Name | Type | Purpose |
|------------|------|---------|
| `merchant_store` | Main table | Store configuration and settings |
| `customer` | Main table | Customer information |
| `shopping_cart` | Main table | Shopping cart data |
| `product` | Main table | Product information |
| `payment_method` | Configuration | Payment method configuration |
| `module_configuration` | Configuration | Module settings |
| `merchant_configuration` | Configuration | Merchant-specific configuration |
| `product_variant` | Related | Product variant details |
| `product_variation` | Related | Product variations |
| `product_variant_group` | Related | Product variant groupings |
| `product_option` | Related | Product options |
| `product_option_value` | Related | Product option values |
| `customer_option` | Related | Customer-specific options |
| `customer_option_value` | Related | Customer option values |
| `language` | Reference | Language settings |
| `country` | Reference | Country information |
| `currency` | Reference | Currency data |
| `tax_class` | Reference | Tax classifications |
| `manufacturer` | Reference | Manufacturer data |
| `product_type` | Reference | Product type categorization |
| `geozone` | Reference | Geographic zones |

### 1.6 High Complexity Methods

The following methods in the call graph have **cyclomatic complexity > 10**:

| Method Name | Cyclomatic Complexity | Essential Complexity | Integration Complexity |
|-------------|----------------------|---------------------|------------------------|
| `getIntegrationModules` | **17** | 2 | 17 |
| `getPopulatedItem` | **15** | 9 | 15 |
| `finalPrice` | **15** | 1 | 15 |
| `buildException` | **14** | 1 | 13 |
| `getAmount` | **14** | 8 | 12 |
| `calculateFinalPrice` | **13** | 2 | 12 |

### 1.7 Key Dependencies

#### Direct Method Calls (Depth 1)
- `getByNick` - Retrieve customer by nickname/code
- `getByCode` - Retrieve shopping cart by code
- `initTransaction` - Initialize payment transaction

#### Core Business Logic Methods
- `getPaymentMethodByCode` - Fetch payment method configuration
- `getPaymentModulesConfigured` - Get configured payment modules
- `getPopulatedShoppingCart` - Build complete shopping cart with items
- `getPopulatedItem` - Populate individual cart item details

#### Data Access Methods
- JPA Entity Operations (SELECT operations)
- Custom repository methods (`findByNick`, `findByCode`, `findBySku`, etc.)

### 1.8 Security Considerations

The endpoint uses:
- **Authentication required** (`/auth/` path prefix)
- Encryption/decryption through `SecretKeySpec` class
- `decrypt` method for sensitive configuration data

---

## 2. Functional Documentation

### 2.1 Business Purpose

This endpoint initializes a payment transaction for a shopping cart. It serves as the entry point for the checkout process, preparing all necessary data before redirecting the customer to the payment gateway.

### 2.2 Functional Flow

1. **Customer & Cart Retrieval**
   - Validates the customer by nickname/code
   - Retrieves the shopping cart associated with the provided code
   - Ensures the cart belongs to the authenticated customer

2. **Shopping Cart Population**
   - Fetches all cart items with complete product details
   - Calculates pricing for each item including:
     - Base product price
     - Product variants and options
     - Tax calculations
     - Final price computation

3. **Payment Configuration**
   - Retrieves payment method by code from the request
   - Loads configured payment modules for the merchant
   - Fetches merchant-specific payment configuration
   - Decrypts sensitive payment credentials if needed

4. **Transaction Initialization**
   - Creates a payment transaction record
   - Associates the transaction with the cart and customer
   - Prepares payment gateway integration data
   - Returns transaction details for frontend processing

### 2.3 Input Parameters

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `payment` | PersistablePayment | Payment details including method code and metadata | Yes |
| `code` | String | Shopping cart code (path variable) | Yes |
| `store` | MerchantStore | Current merchant store context | Yes (implicit) |
| `language` | Language | User's language preference | Yes (implicit) |

### 2.4 Business Rules

1. **Cart Validation**
   - Cart must exist and belong to the requesting customer
   - Cart must contain at least one item
   - All products in cart must be available and active

2. **Payment Method Validation**
   - Payment method must be configured for the merchant
   - Payment module must be active and properly configured
   - Merchant must have valid credentials for the payment gateway

3. **Price Calculation**
   - Prices are calculated with tax considerations
   - Product variants affect final pricing
   - Currency conversion applied based on merchant settings

4. **Error Handling**
   - Comprehensive exception building for various failure scenarios
   - Validation of all entities before transaction creation

### 2.5 Output

Returns a payment initialization response containing:
- Transaction ID
- Payment gateway URL or token
- Order summary with final amounts
- Payment method details
- Redirect parameters for payment gateway

### 2.6 Related Transactions

This endpoint is part of a larger payment workflow:
- **Precedes:** Payment capture/authorization endpoints
- **Follows:** Shopping cart management operations
- **Related:** `api/v1/cart/{}/payment/init/` (non-authenticated variant)

### 2.7 Data Flow Summary

```
Customer Request → Validate Cart → Load Cart Items → Calculate Prices
                                                           ↓
Payment Response ← Initialize Transaction ← Configure Payment ← Load Payment Methods
```

---

## 3. Call Graph Visualization

### 3.1 Complete Call Graph with Complexity Highlighting

The following Mermaid diagram shows the complete call graph for this endpoint. **Nodes highlighted in red** represent methods with **cyclomatic complexity > 10**.

```mermaid
graph TB
    Start["🔵 api/v1/auth/cart/{}/payment/init/<br/>POST Operation<br/>ID: 30571"]
    
    Init["🔴 init<br/>CC: 9<br/>ID: 29714"]
    
    %% Depth 1
    GetByNick["getByNick<br/>CC: N/A<br/>ID: 12642"]
    GetByCode["getByCode<br/>CC: N/A<br/>ID: 13340"]
    InitTransaction["initTransaction<br/>CC: N/A<br/>ID: 29403"]
    
    %% Depth 2
    FindByNick["findByNick<br/>ID: 29250"]
    GetPopulatedCart["🔴 getPopulatedShoppingCart<br/>CC: 9<br/>ID: 13342"]
    FindByCode["findByCode<br/>ID: 29312"]
    GetPaymentMethodByCode["getPaymentMethodByCode<br/>ID: 13000"]
    GetPaymentModulesConfigured["getPaymentModulesConfigured<br/>ID: 29396"]
    
    %% Depth 3
    SelectCustomer["SELECT<br/>JPA: Customer<br/>ID: 30385"]
    GetPopulatedItem["🔴 getPopulatedItem<br/>CC: 15<br/>ID: 29433"]
    SelectCart["SELECT<br/>JPA: ShoppingCart<br/>ID: 30795"]
    GetPaymentMethods["getPaymentMethods<br/>ID: 29394"]
    GetMerchantConfig["getMerchantConfiguration<br/>ID: 13358"]
    Decrypt["decrypt<br/>SecretKeySpec<br/>ID: 29075"]
    
    %% Depth 4
    GetBySku["getBySku<br/>ID: 12522"]
    GetIntegrationModules["🔴 getIntegrationModules<br/>CC: 17<br/>ID: 13381"]
    FindByMerchantStoreAndKey["findByMerchantStoreAndKey<br/>ID: 29315"]
    
    %% Depth 5
    FindBySku["findBySku<br/>ID: 29114"]
    FindByModule["findByModule<br/>ID: 30331"]
    SelectMerchantConfig["SELECT<br/>JPA: MerchantConfig<br/>ID: 30796"]
    
    %% Depth 6
    SelectProduct["SELECT<br/>JPA: Product<br/>ID: 29114"]
    SelectModuleConfig["SELECT<br/>JPA: ModuleConfig<br/>ID: 30810"]
    
    %% Database Tables (Endpoints)
    DB_MerchantStore[("💾 merchant_store<br/>Table ID: 411")]
    DB_Customer[("💾 customer<br/>Table ID: 432")]
    DB_ShoppingCart[("💾 shopping_cart<br/>Table ID: 368")]
    DB_Product[("💾 product<br/>Table ID: 421")]
    DB_ProductVariant[("💾 product_variant<br/>Table ID: 373")]
    DB_CustomerOption[("💾 customer_option<br/>Table ID: 427")]
    DB_CustomerOptionValue[("💾 customer_option_value<br/>Table ID: 424")]
    DB_MerchantConfiguration[("💾 merchant_configuration<br/>Table ID: 414")]
    DB_ModuleConfiguration[("💾 module_configuration<br/>Table ID: 410")]
    DB_Language[("💾 language<br/>Table ID: 417")]
    DB_Country[("💾 country<br/>Table ID: 435")]
    DB_Currency[("💾 currency<br/>Table ID: 433")]
    
    %% Complex methods highlighted with additional styling
    FinalPrice["🔴 finalPrice<br/>CC: 15<br/>ID: 13559"]
    BuildException["🔴 buildException<br/>CC: 14<br/>ID: 11722"]
    GetAmount["🔴 getAmount<br/>CC: 14<br/>ID: 22255"]
    CalculateFinalPrice["🔴 calculateFinalPrice<br/>CC: 13<br/>ID: 13557"]
    GetFinalPrice["🔴 getFinalPrice<br/>CC: 10<br/>ID: 13541"]
    
    %% Main flow connections
    Start --> Init
    Init --> GetByNick
    Init --> GetByCode
    Init --> InitTransaction
    
    %% Depth 2 connections
    GetByNick --> FindByNick
    GetByCode --> GetPopulatedCart
    GetByCode --> FindByCode
    InitTransaction --> GetPaymentMethodByCode
    InitTransaction --> GetPaymentModulesConfigured
    
    %% Depth 3 connections
    FindByNick --> SelectCustomer
    GetPopulatedCart --> GetPopulatedItem
    FindByCode --> SelectCart
    GetPaymentMethodByCode --> GetPaymentMethods
    GetPaymentModulesConfigured --> GetMerchantConfig
    GetPaymentModulesConfigured --> Decrypt
    
    %% Depth 4 connections
    GetPopulatedItem --> GetBySku
    GetPopulatedItem --> FinalPrice
    GetPopulatedItem --> GetAmount
    GetPaymentMethods --> GetIntegrationModules
    GetMerchantConfig --> FindByMerchantStoreAndKey
    
    %% Depth 5 connections
    GetBySku --> FindBySku
    GetIntegrationModules --> FindByModule
    FindByMerchantStoreAndKey --> SelectMerchantConfig
    FinalPrice --> CalculateFinalPrice
    CalculateFinalPrice --> GetFinalPrice
    
    %% Depth 6 connections
    FindBySku --> SelectProduct
    FindByModule --> SelectModuleConfig
    
    %% Database connections
    SelectCustomer --> DB_Customer
    SelectCustomer --> DB_MerchantStore
    SelectCart --> DB_ShoppingCart
    SelectProduct --> DB_Product
    SelectProduct --> DB_ProductVariant
    SelectMerchantConfig --> DB_MerchantConfiguration
    SelectModuleConfig --> DB_ModuleConfiguration
    FindByNick -.-> DB_Language
    FindByNick -.-> DB_Country
    FindByNick -.-> DB_Currency
    GetPopulatedItem -.-> DB_CustomerOption
    GetPopulatedItem -.-> DB_CustomerOptionValue
    
    %% Exception handling
    Init -.-> BuildException
    
    %% Styling
    classDef highComplexity fill:#ff6b6b,stroke:#c92a2a,stroke-width:3px,color:#fff
    classDef mediumComplexity fill:#ffa94d,stroke:#e67700,stroke-width:2px
    classDef database fill:#4dabf7,stroke:#1971c2,stroke-width:2px
    classDef startPoint fill:#51cf66,stroke:#2b8a3e,stroke-width:3px
    
    class Init,GetPopulatedCart highComplexity
    class GetPopulatedItem,GetIntegrationModules,FinalPrice,BuildException,GetAmount,CalculateFinalPrice,GetFinalPrice highComplexity
    class DB_MerchantStore,DB_Customer,DB_ShoppingCart,DB_Product,DB_ProductVariant,DB_CustomerOption,DB_CustomerOptionValue,DB_MerchantConfiguration,DB_ModuleConfiguration,DB_Language,DB_Country,DB_Currency database
    class Start startPoint
```

### 3.2 Legend

- 🔵 **Blue Circle** - API Endpoint (Starting Point)
- 🔴 **Red Box** - High Complexity Method (CC > 10)
- 💾 **Database Cylinder** - Database Table
- **Solid Lines** - Direct method calls
- **Dotted Lines** - Data references or indirect calls
- **CC** - Cyclomatic Complexity

### 3.3 Critical Path Analysis

The critical path through the most complex methods:

1. `api/v1/auth/cart/{}/payment/init/` (Start)
2. → `init` (CC: 9)
3. → `getPopulatedShoppingCart` (CC: 9)
4. → `getPopulatedItem` (CC: 15) **← High Complexity**
5. → `finalPrice` (CC: 15) **← High Complexity**
6. → `calculateFinalPrice` (CC: 13) **← High Complexity**

**Recommendation:** The pricing calculation chain (`getPopulatedItem` → `finalPrice` → `calculateFinalPrice`) contains multiple high-complexity methods and should be prioritized for refactoring and unit testing.

---

## 4. Quality Insights

### 4.1 Detected Issues

The method manifests **2 insight types** (quality concerns). These should be reviewed for:
- Performance optimization opportunities
- Code maintainability improvements
- Security enhancements

### 4.2 Testing Recommendations

**Priority 1 - Critical Path Testing:**
- Test the complete payment initialization flow
- Validate cart retrieval with various cart states
- Test payment method selection and configuration loading

**Priority 2 - Complex Method Testing:**
- Unit tests for `getIntegrationModules` (CC: 17)
- Unit tests for `getPopulatedItem` (CC: 15)
- Unit tests for `finalPrice` (CC: 15)
- Unit tests for `calculateFinalPrice` (CC: 13)

**Priority 3 - Integration Testing:**
- Test database interactions with all 20 tables
- Test encryption/decryption of payment credentials
- Test error handling and exception scenarios

### 4.3 Related Data Graphs

This endpoint is involved in **18 data entity interaction networks**, indicating significant data processing and transformation logic.

---

## 5. Dependencies and Impact

### 5.1 Incoming Dependencies

- **1 caller** uses this endpoint (likely frontend/UI components)

### 5.2 Outgoing Dependencies

- **47 callees** - This method calls 47 other methods
- **20 database tables** - Direct or indirect access
- **Multiple JPA operations** - Entity retrieval and persistence

### 5.3 Change Impact Assessment

**HIGH IMPACT** - Changes to this endpoint affect:
- Payment processing workflow
- Shopping cart functionality
- Price calculation logic
- Multiple downstream payment operations
- Customer checkout experience

**Recommended approach for modifications:**
1. Ensure comprehensive test coverage before changes
2. Use feature flags for incremental rollout
3. Monitor payment success rates after deployment
4. Maintain backward compatibility for API consumers

---

## 6. Notes and References

- **Source Code Location:** `sm-shop/src/main/java/com/salesmanager/shop/store/api/v1/order/OrderPaymentApi.java`
- **Parent Class:** `OrderPaymentApi` (ID: 29357)
- **Programming Language:** Java
- **Framework Version:** Spring MVC with Swagger/OpenAPI annotations
- **Authentication:** Required (`/auth/` prefix)
- **Total Transaction Size:** 643 nodes in complete call graph

---

*Documentation generated from CAST Imaging analysis on January 21, 2026*
