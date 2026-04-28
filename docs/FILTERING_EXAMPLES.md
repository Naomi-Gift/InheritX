# Filtering and Sorting Examples

## Example 1: Plans List with Filters

```typescript
import { useApiQuery } from "@/hooks/useApiQuery";

function PlansListPage() {
  const {
    data: plans,
    loading,
    pagination,
    setFilter,
    setSort,
    setSearch,
    setPage,
    clearFilters,
  } = useApiQuery({
    endpoint: "/api/plans",
    initialParams: {
      filters: { status: "active" },
      sort: [{ field: "created_at", order: "desc" }],
      pagination: { page: 1, limit: 20 },
    },
  });

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      {/* Search */}
      <input
        type="text"
        placeholder="Search plans..."
        onChange={(e) => setSearch(e.target.value)}
      />

      {/* Filters */}
      <select onChange={(e) => setFilter("status", e.target.value)}>
        <option value="">All Statuses</option>
        <option value="active">Active</option>
        <option value="draft">Draft</option>
        <option value="pending">Pending</option>
      </select>

      <select onChange={(e) => setFilter("type", e.target.value)}>
        <option value="">All Types</option>
        <option value="standard">Standard</option>
        <option value="emergency">Emergency</option>
        <option value="business">Business</option>
      </select>

      <button onClick={clearFilters}>Clear Filters</button>

      {/* Sort */}
      <select onChange={(e) => {
        const [field, order] = e.target.value.split(":");
        setSort(field, order as "asc" | "desc");
      }}>
        <option value="created_at:desc">Newest First</option>
        <option value="created_at:asc">Oldest First</option>
        <option value="name:asc">Name A-Z</option>
        <option value="total_assets:desc">Highest Value</option>
      </select>

      {/* Results */}
      <div>
        {plans.map((plan) => (
          <div key={plan.id}>
            <h3>{plan.name}</h3>
            <p>Status: {plan.status}</p>
            <p>Assets: ${plan.total_assets}</p>
          </div>
        ))}
      </div>

      {/* Pagination */}
      <div>
        <button
          onClick={() => setPage(pagination.page - 1)}
          disabled={pagination.page === 1}
        >
          Previous
        </button>
        <span>
          Page {pagination.page} of {pagination.totalPages}
        </span>
        <button
          onClick={() => setPage(pagination.page + 1)}
          disabled={pagination.page === pagination.totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

## Example 2: Claims with Multiple Filters

```typescript
function ClaimsListPage() {
  const { data: claims, setFilter, setSort } = useApiQuery({
    endpoint: "/api/claims",
  });

  return (
    <div>
      {/* Multiple status filter */}
      <div>
        <label>
          <input
            type="checkbox"
            onChange={(e) => {
              if (e.target.checked) {
                setFilter("status", ["pending", "processing"]);
              }
            }}
          />
          Show Pending & Processing
        </label>
      </div>

      {/* Sort by amount */}
      <button onClick={() => setSort("amount", "desc")}>
        Highest Amount First
      </button>

      {claims.map((claim) => (
        <div key={claim.id}>
          <p>{claim.beneficiary_name}</p>
          <p>${claim.amount}</p>
          <p>{claim.status}</p>
        </div>
      ))}
    </div>
  );
}
```

## Example 3: Direct API Call

```typescript
import { buildQueryString } from "@/lib/api/filtering";

async function fetchFilteredPlans() {
  const queryString = buildQueryString({
    filters: {
      status: "active",
      type: "standard",
    },
    sort: [
      { field: "created_at", order: "desc" },
      { field: "name", order: "asc" },
    ],
    search: "family",
    pagination: { page: 1, limit: 20 },
  });

  const response = await fetch(`/api/plans?${queryString}`);
  const result = await response.json();

  console.log("Plans:", result.data);
  console.log("Total:", result.pagination.total);
}
```

## Example 4: Server-Side Filtering

```typescript
// In a Next.js API route or server component
import { parseQueryParams, applyQueryParams } from "@/lib/api/filtering";

export async function GET(request: Request) {
  const url = new URL(request.url);
  const params = parseQueryParams(url.searchParams);

  // Get data from database
  const allPlans = await db.plans.findMany();

  // Apply filtering, sorting, pagination
  const result = applyQueryParams(allPlans, params, [
    "name",
    "status",
    "type",
  ]);

  return Response.json({
    status: "ok",
    data: result.data,
    pagination: {
      total: result.total,
      page: result.page,
      limit: result.limit,
      totalPages: result.totalPages,
    },
  });
}
```

## Example 5: Advanced Search

```typescript
function AdvancedSearchPage() {
  const { data, setFilter, setSearch, queryParams } = useApiQuery({
    endpoint: "/api/messages",
  });

  const handleAdvancedSearch = () => {
    // Combine multiple filters
    setFilter("status", "scheduled");
    setFilter("priority", "high");
    setSearch("emergency");
  };

  return (
    <div>
      <button onClick={handleAdvancedSearch}>
        Find High Priority Emergency Messages
      </button>

      {/* Show active filters */}
      <div>
        Active Filters:
        {Object.entries(queryParams.filters || {}).map(([key, value]) => (
          <span key={key}>
            {key}: {String(value)}
          </span>
        ))}
      </div>

      {data.map((message) => (
        <div key={message.id}>{message.title}</div>
      ))}
    </div>
  );
}
```

## URL Examples

### Filter by status
```
/api/plans?status=active
```

### Multiple filters
```
/api/plans?status=active&type=standard
```

### Sort by multiple fields
```
/api/plans?sort=status:asc,created_at:desc
```

### Search with filters
```
/api/plans?status=active&search=family&page=1&limit=20
```

### Complex query
```
/api/claims?status=pending&status=processing&sort=amount:desc&search=john&page=2&limit=10
```
