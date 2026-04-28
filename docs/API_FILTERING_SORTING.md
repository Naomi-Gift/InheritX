# API Filtering and Sorting Documentation

## Overview
All list endpoints now support filtering, sorting, searching, and pagination through query parameters.

## Query Parameters

### Filtering
Filter results by field values:
```
GET /api/plans?status=active&type=standard
```

Multiple values (OR logic):
```
GET /api/plans?status=active&status=pending
```

### Sorting
Sort by one or more fields:
```
GET /api/plans?sort=created_at:desc
```

Multiple sort fields:
```
GET /api/plans?sort=status:asc,created_at:desc
```

Sort orders: `asc` (ascending), `desc` (descending)

### Searching
Search across multiple fields:
```
GET /api/plans?search=family
```

### Pagination
Control page size and number:
```
GET /api/plans?page=2&limit=20
```

Default: `page=1&limit=10`

### Combined Example
```
GET /api/plans?status=active&sort=created_at:desc&search=trust&page=1&limit=20
```

## Response Format

All list endpoints return:
```json
{
  "status": "ok",
  "data": [...],
  "pagination": {
    "total": 50,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  },
  "filters": {
    "status": "active"
  },
  "sort": [
    { "field": "created_at", "order": "desc" }
  ]
}
```

## Supported Endpoints

### Plans
```
GET /api/plans
```
Filterable fields: `status`, `type`, `owner_address`
Searchable fields: `name`, `status`, `type`, `owner_address`
Sortable fields: `created_at`, `updated_at`, `name`, `total_assets`, `beneficiaries_count`

### Claims
```
GET /api/claims
```
Filterable fields: `status`, `claim_type`, `plan_id`
Searchable fields: `beneficiary_name`, `status`, `claim_type`
Sortable fields: `submitted_at`, `updated_at`, `amount`, `status`

### Messages
```
GET /api/messages
```
Filterable fields: `status`, `priority`, `vault_id`
Searchable fields: `title`, `status`, `priority`
Sortable fields: `created_at`, `updated_at`, `unlock_at`, `title`

### Emergency Contacts
```
GET /api/emergency/contacts/:planId
```
Filterable fields: `relationship`, `verified`
Searchable fields: `name`, `email`, `relationship`
Sortable fields: `added_at`, `name`

### Will Documents
```
GET /api/plans/:planId/will/documents
```
Filterable fields: `status`, `template_used`, `signed`
Searchable fields: `template_used`, `status`, `filename`
Sortable fields: `generated_at`, `version`

### Audit Logs
```
GET /api/emergency/audit-logs
```
Filterable fields: `action`, `entity_type`, `performed_by`
Searchable fields: `action`, `entity_type`, `performed_by`
Sortable fields: `timestamp`, `action`

## Using in React Components

### With useApiQuery Hook
```typescript
import { useApiQuery } from "@/hooks/useApiQuery";

function PlansList() {
  const {
    data,
    loading,
    pagination,
    setFilter,
    setSort,
    setSearch,
    setPage,
  } = useApiQuery({
    endpoint: "/api/plans",
    initialParams: {
      filters: { status: "active" },
      sort: [{ field: "created_at", order: "desc" }],
      pagination: { page: 1, limit: 20 },
    },
  });

  return (
    <div>
      <input onChange={(e) => setSearch(e.target.value)} />
      <select onChange={(e) => setFilter("status", e.target.value)}>
        <option value="active">Active</option>
        <option value="draft">Draft</option>
      </select>
      {/* Render data */}
    </div>
  );
}
```

### Direct Fetch
```typescript
import { buildQueryString } from "@/lib/api/filtering";

const queryString = buildQueryString({
  filters: { status: "active" },
  sort: [{ field: "created_at", order: "desc" }],
  pagination: { page: 1, limit: 20 },
});

const response = await fetch(`/api/plans?${queryString}`);
const data = await response.json();
```

## Filter Operators

### Exact Match
```
?status=active
```

### Wildcard Match
```
?name=*trust*  (contains "trust")
?name=family*  (starts with "family")
?name=*plan    (ends with "plan")
```

### Multiple Values (OR)
```
?status=active&status=pending
```

## Best Practices

1. **Always paginate** - Use reasonable limit values (10-100)
2. **Index sort fields** - Ensure database indexes on commonly sorted fields
3. **Limit search scope** - Specify searchable fields to improve performance
4. **Cache results** - Consider caching for frequently accessed filtered data
5. **Validate inputs** - Always validate and sanitize query parameters

## Performance Considerations

- Filtering is applied before sorting
- Search is applied after filtering
- Pagination is applied last
- Use database-level filtering for large datasets
- Consider implementing cursor-based pagination for very large datasets

## Error Handling

Invalid parameters return 400 Bad Request:
```json
{
  "status": "error",
  "message": "Invalid sort field: invalid_field"
}
```

## Testing

Run filtering tests:
```bash
npm test tests/api/filtering.test.ts
```

## Migration Guide

### Before
```typescript
const response = await fetch("/api/plans");
const plans = await response.json();
```

### After
```typescript
const response = await fetch("/api/plans?status=active&sort=created_at:desc&page=1&limit=20");
const { data, pagination } = await response.json();
```
