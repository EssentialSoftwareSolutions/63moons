# RBAC Integration Example

This document shows a practical example of integrating the RBAC system into an existing Angular project.

## Scenario: E-commerce Admin Panel

Let's say you have an existing e-commerce admin panel and want to add role-based access control.

### Step 1: Define Your Roles and Permissions

```typescript
// Define your business roles
const roles = [
  {
    name: 'super_admin',
    permissions: ['*:*'] // All permissions
  },
  {
    name: 'admin',
    permissions: [
      'users:read', 'users:write', 'users:delete',
      'products:read', 'products:write', 'products:delete',
      'orders:read', 'orders:write',
      'reports:read'
    ]
  },
  {
    name: 'manager',
    permissions: [
      'products:read', 'products:write',
      'orders:read', 'orders:write',
      'reports:read'
    ]
  },
  {
    name: 'support',
    permissions: [
      'users:read',
      'orders:read', 'orders:write'
    ]
  },
  {
    name: 'viewer',
    permissions: [
      'products:read',
      'orders:read',
      'reports:read'
    ]
  }
];
```

### Step 2: Update Your Existing Routes

```typescript
// Before: No protection
const routes: Routes = [
  { path: 'products', component: ProductsComponent },
  { path: 'orders', component: OrdersComponent },
  { path: 'users', component: UsersComponent },
  { path: 'reports', component: ReportsComponent }
];

// After: With RBAC protection
const routes: Routes = [
  { path: 'login', component: LoginComponent },
  { path: 'unauthorized', component: UnauthorizedComponent },
  {
    path: '',
    canActivate: [AuthGuard],
    children: [
      {
        path: 'products',
        component: ProductsComponent,
        canActivate: [RoleGuard],
        data: { permissions: ['products:read'] }
      },
      {
        path: 'products/edit',
        component: ProductEditComponent,
        canActivate: [RoleGuard],
        data: { permissions: ['products:write'] }
      },
      {
        path: 'orders',
        component: OrdersComponent,
        canActivate: [RoleGuard],
        data: { permissions: ['orders:read'] }
      },
      {
        path: 'users',
        component: UsersComponent,
        canActivate: [RoleGuard],
        data: { 
          permissions: ['users:read'],
          roles: ['admin', 'super_admin'] 
        }
      },
      {
        path: 'reports',
        component: ReportsComponent,
        canActivate: [RoleGuard],
        data: { permissions: ['reports:read'] }
      }
    ]
  }
];
```

### Step 3: Update Your Components

```typescript
// products.component.ts - Before
export class ProductsComponent {
  products: Product[] = [];

  constructor(private productService: ProductService) {}

  deleteProduct(id: string) {
    this.productService.delete(id).subscribe();
  }
}

// products.component.ts - After
export class ProductsComponent {
  products: Product[] = [];

  constructor(
    private productService: ProductService,
    private permissionService: PermissionService
  ) {}

  deleteProduct(id: string) {
    if (this.canDeleteProducts()) {
      this.productService.delete(id).subscribe();
    }
  }

  canDeleteProducts(): boolean {
    return this.permissionService.canAccess('products', 'delete');
  }

  canEditProducts(): boolean {
    return this.permissionService.canAccess('products', 'write');
  }
}
```

### Step 4: Update Your Templates

```html
<!-- products.component.html - Before -->
<div class="products-container">
  <h2>Products</h2>
  <button class="btn btn-primary" routerLink="/products/new">
    Add Product
  </button>
  
  <table class="table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Price</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let product of products">
        <td>{{ product.name }}</td>
        <td>{{ product.price | currency }}</td>
        <td>
          <button class="btn btn-sm btn-primary" [routerLink]="['/products/edit', product.id]">
            Edit
          </button>
          <button class="btn btn-sm btn-danger" (click)="deleteProduct(product.id)">
            Delete
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</div>

<!-- products.component.html - After -->
<div class="products-container">
  <h2>Products</h2>
  
  <!-- Only show Add button if user can create products -->
  <button 
    *hasPermission="'products:write'" 
    class="btn btn-primary" 
    routerLink="/products/new">
    Add Product
  </button>
  
  <table class="table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Price</th>
        <th *hasPermission="['products:write', 'products:delete']">Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let product of products">
        <td>{{ product.name }}</td>
        <td>{{ product.price | currency }}</td>
        <td *hasPermission="['products:write', 'products:delete']">
          <button 
            *hasPermission="'products:write'"
            class="btn btn-sm btn-primary" 
            [routerLink]="['/products/edit', product.id]">
            Edit
          </button>
          <button 
            *hasPermission="'products:delete'"
            class="btn btn-sm btn-danger" 
            (click)="deleteProduct(product.id)">
            Delete
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

### Step 5: Add Navigation Protection

```html
<!-- navigation.component.html - Before -->
<nav class="navbar">
  <ul class="nav-menu">
    <li><a routerLink="/products">Products</a></li>
    <li><a routerLink="/orders">Orders</a></li>
    <li><a routerLink="/users">Users</a></li>
    <li><a routerLink="/reports">Reports</a></li>
  </ul>
</nav>

<!-- navigation.component.html - After -->
<nav class="navbar">
  <ul class="nav-menu">
    <li *hasPermission="'products:read'">
      <a routerLink="/products">Products</a>
    </li>
    <li *hasPermission="'orders:read'">
      <a routerLink="/orders">Orders</a>
    </li>
    <li *hasRole="['admin', 'super_admin']">
      <a routerLink="/users">Users</a>
    </li>
    <li *hasPermission="'reports:read'">
      <a routerLink="/reports">Reports</a>
    </li>
  </ul>
  
  <!-- User info and logout -->
  <div class="user-menu" *ngIf="currentUser">
    <span>{{ currentUser.firstName }} {{ currentUser.lastName }}</span>
    <span class="role-badge" *ngFor="let role of currentUser.roles">
      {{ role.name }}
    </span>
    <button class="btn btn-sm" (click)="logout()">Logout</button>
  </div>
</nav>
```

### Step 6: Handle Different User Experiences

```typescript
// dashboard.component.ts
export class DashboardComponent implements OnInit {
  dashboardItems: DashboardItem[] = [];

  constructor(private permissionService: PermissionService) {}

  ngOnInit() {
    this.loadDashboardItems();
  }

  private loadDashboardItems() {
    const allItems = [
      {
        title: 'Products',
        icon: '📦',
        route: '/products',
        permission: 'products:read'
      },
      {
        title: 'Orders',
        icon: '📋',
        route: '/orders',
        permission: 'orders:read'
      },
      {
        title: 'Users',
        icon: '👥',
        route: '/users',
        permission: 'users:read'
      },
      {
        title: 'Reports',
        icon: '📊',
        route: '/reports',
        permission: 'reports:read'
      }
    ];

    // Filter items based on user permissions
    this.dashboardItems = allItems.filter(item => 
      this.permissionService.hasPermission(item.permission)
    );
  }
}
```

### Step 7: Backend Integration

```typescript
// auth.service.ts - Update for your API
export class AuthService {
  private readonly API_URL = 'https://your-api.com/api';

  login(credentials: LoginRequest): Observable<LoginResponse> {
    return this.http.post<LoginResponse>(`${this.API_URL}/auth/login`, credentials);
  }

  // Your backend should return user data with roles and permissions
  // Example response:
  // {
  //   "user": {
  //     "id": "123",
  //     "username": "john.doe",
  //     "roles": [
  //       {
  //         "name": "manager",
  //         "permissions": [
  //           { "resource": "products", "action": "read" },
  //           { "resource": "products", "action": "write" }
  //         ]
  //       }
  //     ]
  //   },
  //   "token": "jwt-token",
  //   "refreshToken": "refresh-token"
  // }
}
```

## Migration Checklist

- [ ] Copy RBAC files to your project
- [ ] Update app.module.ts with RBAC imports
- [ ] Define your roles and permissions
- [ ] Update routing configuration
- [ ] Add guards to protected routes
- [ ] Update component logic for permission checks
- [ ] Add directives to templates
- [ ] Update navigation menus
- [ ] Test with different user roles
- [ ] Update backend API integration
- [ ] Add error handling for unauthorized access
- [ ] Test token refresh functionality

## Testing Different Roles

Create test users with different roles to verify the system works correctly:

```typescript
// Mock users for testing
const testUsers = [
  {
    username: 'admin',
    roles: ['admin'],
    // Should see all features
  },
  {
    username: 'manager',
    roles: ['manager'],
    // Should see products, orders, reports
  },
  {
    username: 'support',
    roles: ['support'],
    // Should see users (read-only), orders
  },
  {
    username: 'viewer',
    roles: ['viewer'],
    // Should see read-only access to most features
  }
];
```

This example shows how to gradually integrate RBAC into your existing application with minimal disruption to your current codebase.
