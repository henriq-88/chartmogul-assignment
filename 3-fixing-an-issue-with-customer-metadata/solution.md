# Original Code

```ruby
def custom_params
  {
    'Account owner' => "#{owner.name} #{owner.email}",
    'Account status' => account_details.account_status,
    'Active customers' => account_details.active_customer_count,
    'Billing system(s)' => billing_connectors.presence || ['None configured'],
    'Import status' => main_connector.try(:import_status).try(:humanize) || 'Never imported',
    'Last active' => @account.last_active_at,
    'Revenue recognition access' => @account.rev_rec_enabled?,
    'Sample Data Present?' => sample_data_present?
  }.reject { |_k, v| v.blank? }
end
```

# Solution

```ruby
def custom_params
  {
    'Account owner name' => owner.name,
    'Account id' => @account.id,
    'Account provider type' => 'email',
    'Account provider id' => owner.email,
    'Account owner' => "#{owner.name} #{owner.email}",
    'Account status' => account_details.account_status,
    'Active customers' => account_details.active_customer_count,
    'Billing system(s)' => billing_connectors.presence || ['None configured'],
    'Import status' => main_connector.try(:import_status).try(:humanize) || [ `No imports in progess` ],
    'Last active' => @account.last_active_at,
    'Revenue recognition access' => @account.rev_rec_enabled?,
    'Sample Data Present?' => sample_data_present?
  }.reject { |_k, v| v.blank? }
end
```

## Explanation

### Splitting of `Account name`

To be able to quickly and efficiently query data based on users' email or name, it would be better to split up the `Account owner` field into different fields so they can be indexed seperately, without having to do text contains search on each entry.

- `Account name`
- `Account id` (e.g. `229f7854-1b2a-11ee-be56-0242ac120002`)
  - A unified structured field for internal ids.
- `Account provider type` (e.g. `email`, `twitter`, `facebook`, `phone`)
  - Allow users to be able to connect with various providers.
- `Account provider id` (e.g. `user@company.com`, `+1234567890`, `12873912381`)
  - The related provider's id, in whatever format they have.

When splitting the `Account owner` field into smaller fields, for the purpose of backwards compatibility, keep the field for a period of time, until the rest of the code has been migrated and the field finally gets sunset.

### Allow Parallel Imports

Another possible problem is that a user might want to import from multiple sources at the same time, and with the current metadata structure only one import status can be reported at a time.

If users should be allowed to import in parallel the `Import status` field could change to become an array of statuses.
