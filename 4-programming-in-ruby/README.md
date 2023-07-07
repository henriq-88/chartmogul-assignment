# Provided Dataset

```ruby
payment_events = [
  { "event": "subscription", "account": "A001", "date": "2020-01-15" },
  { "event": "subscription", "account": "B001", "date": "2020-01-23" },
  { "event": "subscription", "account": "C001", "date": "2020-01-25" },
  { "event": "cancellation", "account": "B001", "date": "2020-02-23" },
  { "event": "reactivation", "account": "B001", "date": "2020-04-01" },
  { "event": "cancellation", "account": "B001", "date": "2020-04-03" },
  { "event": "reactivation", "account": "B001", "date": "2020-05-12" },
  { "event": "cancellation", "account": "B001", "date": "2020-06-18" },
  { "event": "cancellation", "account": "C001", "date": "2020-03-10" },
  { "event": "reactivation", "account": "C001", "date": "2020-05-17" }
]
```

# Solution

## 0. Group Payment Events by Account

```ruby
accounts = []

class Account
  attr_reader :id, :payment_events

  def initialize(id)
    @id = id
    @payment_events = []
  end

  def add_payment_event(event)
    payment_events.push(event)
  end
end

def group_payment_events_by_account(accounts, payment_events)
  for payment_event in payment_events do
    account_id = payment_event[:account]
    account = accounts.find { |account| account.id == payment_event[:account] }
    if account.nil? then
      account = Account.new(account_id)
      accounts.push(account)
    end
    account.add_payment_event(payment_event)
  end
end

group_payment_events_by_account(accounts, payment_events)
```

## 1. Which Accounts Have At Least One Reactivation?

```ruby
def get_accounts_by_payment_event_status(accounts, status)
  # puts "\naccounts with a payment event status of \"#{status}\":"
  return accounts.select { |account| account.payment_events.select { |payment_event| payment_event[:event] == status }.length > 0 }.map { |account| account.id }
end

puts get_accounts_by_payment_event_status(accounts, 'reactivation')
```

### Result

```
B001
C001
```

## 2. As of April 2nd, 2020 Which Accounts Have Active Subscriptions?

```ruby
def get_accounts_as_of_date_by_payment_event_statuses(accounts, date, statuses)
  # puts "\naccounts as of #{date} with last status of #{statuses.map { |status| "\"#{status}\"" }.join(' or ')}:"
  return accounts.select { |account| statuses.include? account.payment_events.select { |payment_event| payment_event[:date] <= date }.last[:event] }.map { |account| account.id }
end

puts get_accounts_as_of_date_by_payment_event_statuses(accounts, '2020-04-02', ['subscription', 'reactivation'])
```

### Result

```
A001
B001
```
