# Backend API Structure
## Warehouse Management System - REST API Specification

### 🏗️ **API Architecture Overview**

```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway / Load Balancer              │
├─────────────────────────────────────────────────────────────┤
│                    Authentication Middleware                │
├─────────────────────────────────────────────────────────────┤
│                    Rate Limiting & CORS                     │
├─────────────────────────────────────────────────────────────┤
│  Auth Service  │  Inventory Service  │  Order Service       │
│  User Service  │  Product Service    │  Report Service      │
│  Audit Service │  Warehouse Service  │  Integration Service │
├─────────────────────────────────────────────────────────────┤
│                    Database Layer (PostgreSQL)              │
│                    Redis Cache Layer                        │
└─────────────────────────────────────────────────────────────┘
```

### 🔐 **Authentication & Authorization**

#### Auth Endpoints
```
POST   /api/v1/auth/login              # User login
POST   /api/v1/auth/logout             # User logout  
POST   /api/v1/auth/refresh            # Refresh JWT token
POST   /api/v1/auth/forgot-password    # Password reset request
POST   /api/v1/auth/reset-password     # Reset password with token
GET    /api/v1/auth/me                 # Get current user info
```

#### JWT Token Structure
```json
{
  "user_id": "uuid",
  "username": "string",
  "role": "admin|manager|staff|viewer",
  "warehouse_id": "uuid",
  "permissions": ["read:inventory", "write:orders", "..."],
  "exp": "timestamp",
  "iat": "timestamp"
}
```

---

### 👥 **User Management API**

```
GET    /api/v1/users                   # List all users (admin only)
POST   /api/v1/users                   # Create new user (admin only)
GET    /api/v1/users/:id               # Get user by ID
PUT    /api/v1/users/:id               # Update user (admin/self)
DELETE /api/v1/users/:id               # Deactivate user (admin only)
PUT    /api/v1/users/:id/password      # Change password (self)
PUT    /api/v1/users/:id/role          # Change user role (admin only)
```

#### User Model
```json
{
  "id": "uuid",
  "username": "string",
  "email": "string",
  "first_name": "string",
  "last_name": "string",
  "phone": "string",
  "role": "admin|manager|staff|viewer",
  "is_active": "boolean",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### 📦 **Product Management API**

```
GET    /api/v1/products                # List all products (paginated)
POST   /api/v1/products                # Create new product
GET    /api/v1/products/:id            # Get product by ID
PUT    /api/v1/products/:id            # Update product
DELETE /api/v1/products/:id            # Deactivate product
GET    /api/v1/products/search         # Search products by name/SKU
GET    /api/v1/products/barcode/:code  # Get product by barcode
POST   /api/v1/products/import         # Bulk import products (CSV/Excel)
GET    /api/v1/products/export         # Export products to CSV/Excel
```

#### Product Model
```json
{
  "id": "uuid",
  "sku": "string",
  "name": "string",
  "description": "string",
  "category": {
    "id": "uuid",
    "name": "string"
  },
  "supplier": {
    "id": "uuid",
    "name": "string"
  },
  "cost_price": "decimal",
  "selling_price": "decimal",
  "unit": "pcs|kg|liter|box|meter",
  "barcode": "string",
  "weight": "decimal",
  "dimensions": "string",
  "is_active": "boolean",
  "created_at": "timestamp",
  "updated_at": "timestamp"
}
```

---

### 📊 **Inventory Management API**

```
GET    /api/v1/inventory               # List inventory (with filters)
GET    /api/v1/inventory/:id           # Get inventory item details
PUT    /api/v1/inventory/:id           # Update stock levels (admin/manager)
GET    /api/v1/inventory/low-stock     # Get low stock items
GET    /api/v1/inventory/movements     # Get inventory movements history
POST   /api/v1/inventory/adjustments  # Create stock adjustment
GET    /api/v1/inventory/adjustments  # List stock adjustments
PUT    /api/v1/inventory/adjustments/:id/approve  # Approve adjustment
```

#### Inventory Model
```json
{
  "id": "uuid",
  "product": {
    "id": "uuid",
    "sku": "string",
    "name": "string"
  },
  "warehouse": {
    "id": "uuid",
    "name": "string"
  },
  "location": {
    "id": "uuid", 
    "location_code": "string"
  },
  "quantity": "integer",
  "reserved_quantity": "integer", 
  "available_quantity": "integer",
  "min_stock_level": "integer",
  "max_stock_level": "integer",
  "average_cost": "decimal",
  "stock_status": "OK|LOW|OUT_OF_STOCK",
  "last_movement": "timestamp"
}
```

---

### 🏢 **Warehouse & Location API**

```
GET    /api/v1/warehouses              # List all warehouses
POST   /api/v1/warehouses              # Create new warehouse
GET    /api/v1/warehouses/:id          # Get warehouse details
PUT    /api/v1/warehouses/:id          # Update warehouse
GET    /api/v1/warehouses/:id/locations # Get warehouse locations
POST   /api/v1/locations               # Create new location
GET    /api/v1/locations/:id           # Get location details
PUT    /api/v1/locations/:id           # Update location
```

---

### 🛒 **Purchase Order API (Inbound)**

```
GET    /api/v1/purchase-orders         # List purchase orders
POST   /api/v1/purchase-orders         # Create new PO
GET    /api/v1/purchase-orders/:id     # Get PO details
PUT    /api/v1/purchase-orders/:id     # Update PO
DELETE /api/v1/purchase-orders/:id     # Cancel PO
PUT    /api/v1/purchase-orders/:id/send # Send PO to supplier
```

#### Purchase Order Model
```json
{
  "id": "uuid",
  "po_number": "string",
  "supplier": {
    "id": "uuid",
    "name": "string"
  },
  "order_date": "date",
  "expected_date": "date",
  "status": "draft|sent|confirmed|partially_received|received|cancelled",
  "subtotal": "decimal",
  "tax_amount": "decimal", 
  "total_amount": "decimal",
  "items": [
    {
      "product_id": "uuid",
      "product_name": "string",
      "ordered_quantity": "integer",
      "received_quantity": "integer",
      "unit_price": "decimal",
      "total_price": "decimal"
    }
  ],
  "notes": "string",
  "created_by": "string",
  "created_at": "timestamp"
}
```

---

### 📥 **Goods Receipt API (GRN)**

```
GET    /api/v1/receipts                # List goods receipts
POST   /api/v1/receipts                # Create new GRN
GET    /api/v1/receipts/:id            # Get GRN details  
PUT    /api/v1/receipts/:id            # Update GRN
PUT    /api/v1/receipts/:id/receive    # Mark items as received
PUT    /api/v1/receipts/:id/quality-check # Quality check results
PUT    /api/v1/receipts/:id/put-away   # Put away to locations
```

#### Goods Receipt Model
```json
{
  "id": "uuid",
  "grn_number": "string",
  "purchase_order": {
    "id": "uuid",
    "po_number": "string"
  },
  "warehouse_id": "uuid",
  "receipt_date": "date",
  "status": "draft|received|quality_check|put_away|completed",
  "items": [
    {
      "product_id": "uuid",
      "quantity_received": "integer",
      "quantity_accepted": "integer", 
      "quantity_rejected": "integer",
      "rejection_reason": "string",
      "location_id": "uuid",
      "batch_number": "string",
      "expiry_date": "date"
    }
  ],
  "received_by": "string",
  "notes": "string"
}
```

---

### 🛍️ **Sales Order API (Outbound)**

```
GET    /api/v1/sales-orders            # List sales orders
POST   /api/v1/sales-orders            # Create new sales order
GET    /api/v1/sales-orders/:id        # Get sales order details
PUT    /api/v1/sales-orders/:id        # Update sales order
DELETE /api/v1/sales-orders/:id        # Cancel sales order
PUT    /api/v1/sales-orders/:id/confirm # Confirm order
PUT    /api/v1/sales-orders/:id/allocate # Allocate inventory
```

#### Sales Order Model
```json
{
  "id": "uuid",
  "order_number": "string",
  "customer": {
    "id": "uuid",
    "name": "string"
  },
  "order_date": "date",
  "requested_date": "date", 
  "priority": "low|medium|high|urgent",
  "status": "draft|confirmed|allocated|picking|packed|shipped|delivered|cancelled",
  "subtotal": "decimal",
  "tax_amount": "decimal",
  "shipping_cost": "decimal",
  "total_amount": "decimal",
  "items": [
    {
      "product_id": "uuid",
      "product_name": "string",
      "ordered_quantity": "integer",
      "allocated_quantity": "integer",
      "picked_quantity": "integer", 
      "shipped_quantity": "integer",
      "unit_price": "decimal",
      "total_price": "decimal"
    }
  ],
  "notes": "string"
}
```

---

### 🚚 **Shipment API (Delivery Order)**

```
GET    /api/v1/shipments               # List shipments
POST   /api/v1/shipments               # Create new shipment
GET    /api/v1/shipments/:id           # Get shipment details
PUT    /api/v1/shipments/:id           # Update shipment
PUT    /api/v1/shipments/:id/pick      # Pick items for shipment
PUT    /api/v1/shipments/:id/pack      # Pack shipment
PUT    /api/v1/shipments/:id/ship      # Mark as shipped
```

---

### 🏷️ **Category & Supplier API**

```
GET    /api/v1/categories              # List categories (tree structure)
POST   /api/v1/categories              # Create category
PUT    /api/v1/categories/:id          # Update category
DELETE /api/v1/categories/:id          # Delete category

GET    /api/v1/suppliers               # List suppliers
POST   /api/v1/suppliers               # Create supplier
GET    /api/v1/suppliers/:id           # Get supplier details
PUT    /api/v1/suppliers/:id           # Update supplier
DELETE /api/v1/suppliers/:id           # Deactivate supplier
```

---

### 📱 **Mobile API (Barcode/QR Operations)**

```
POST   /api/v1/mobile/scan             # Process barcode/QR scan
GET    /api/v1/mobile/product/:barcode # Get product by barcode
POST   /api/v1/mobile/receive          # Mobile goods receipt
POST   /api/v1/mobile/pick             # Mobile picking
POST   /api/v1/mobile/count            # Mobile cycle count
```

---

### 📈 **Reporting & Analytics API**

```
GET    /api/v1/reports/dashboard       # Dashboard summary data
GET    /api/v1/reports/inventory       # Inventory reports
GET    /api/v1/reports/movements       # Stock movement reports  
GET    /api/v1/reports/orders          # Order reports
GET    /api/v1/reports/abc-analysis    # ABC analysis
GET    /api/v1/reports/turnover        # Inventory turnover
POST   /api/v1/reports/custom          # Custom report generation
GET    /api/v1/reports/export/:id      # Export report to PDF/Excel
```

#### Dashboard Model
```json
{
  "summary": {
    "total_products": "integer",
    "total_stock_value": "decimal",
    "low_stock_items": "integer",
    "pending_orders": "integer"
  },
  "recent_activities": [
    {
      "type": "inbound|outbound|adjustment",
      "description": "string", 
      "timestamp": "timestamp",
      "user": "string"
    }
  ],
  "charts": {
    "stock_levels": {...},
    "order_trends": {...},
    "top_products": {...}
  }
}
```

---

### 🔍 **Audit & Monitoring API**

```
GET    /api/v1/audit/logs              # Get audit logs (admin only)
GET    /api/v1/audit/user/:id          # Get user activity logs
GET    /api/v1/audit/product/:id       # Get product change history
GET    /api/v1/health                  # System health check
GET    /api/v1/metrics                 # System metrics
```

---

### 🔗 **Integration API (External Systems)**

```
POST   /api/v1/webhook/orders          # Receive orders from e-commerce
POST   /api/v1/webhook/inventory       # Sync inventory with ERP
GET    /api/v1/sync/products           # Sync products to/from ERP
POST   /api/v1/sync/customers          # Sync customer data
```

---

### 📊 **API Response Formats**

#### Success Response
```json
{
  "success": true,
  "data": {...},
  "message": "Operation completed successfully",
  "timestamp": "2024-01-01T10:00:00Z"
}
```

#### Error Response  
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  },
  "timestamp": "2024-01-01T10:00:00Z"
}
```

#### Paginated Response
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false
  }
}
```

---

### 🔒 **Permission Matrix**

| Endpoint | Admin | Manager | Staff | Customer |
|----------|-------|---------|-------|----------|
| **Auth** | ✅ | ✅ | ✅ | ✅ |
| **Users CRUD** | ✅ | ❌ | ❌ | ❌ |
| **Products CRUD** | ✅ | ✅ | 👁️ | ❌ |
| **Inventory View** | ✅ | ✅ | ✅ | ❌ |
| **Inventory Update** | ✅ | ✅ | ❌ | ❌ |
| **Purchase Orders** | ✅ | ✅ | ❌ | ❌ |
| **Goods Receipt** | ✅ | ✅ | ✅ | ❌ |
| **Sales Orders** | ✅ | ✅ | ✅ | ✅ (own) |
| **Shipments** | ✅ | ✅ | ✅ | 👁️ (own) |
| **Reports** | ✅ | ✅ | 📊 (limited) | ❌ |
| **Mobile Operations** | ✅ | ✅ | ✅ | ❌ |
| **Audit Logs** | ✅ | ❌ | ❌ | ❌ |

**Legend**: ✅ Full Access, 👁️ Read Only, 📊 Limited Access, ❌ No Access

---

### ⚡ **Performance & Caching Strategy**

- **Redis Cache**: Product info, inventory levels, user sessions
- **Database Indexing**: Optimized queries for inventory and orders
- **Pagination**: All list endpoints support pagination
- **Rate Limiting**: API rate limits per user role
- **Connection Pooling**: Database connection optimization
- **Async Processing**: Heavy operations (reports, imports) via queue