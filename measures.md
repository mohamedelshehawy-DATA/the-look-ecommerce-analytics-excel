# DAX Measures

## Funnel (events)

```
Sessions := DISTINCTCOUNT(events[session_id])

Cart Sessions := CALCULATE(DISTINCTCOUNT(events[session_id]), events[event_type] = "cart")

Purchase Sessions := CALCULATE(DISTINCTCOUNT(events[session_id]), events[event_type] = "purchase")

Session to Cart := DIVIDE([Cart Sessions], [Sessions])
Cart to Purchase := DIVIDE([Purchase Sessions], [Cart Sessions])
Conversion Rate := DIVIDE([Purchase Sessions], [Sessions])
Cart Abandonment := 1 - [Cart to Purchase]
```

Sessions are counted with DISTINCTCOUNT, not COUNTROWS, because one session has many events.

## Revenue and profit (order_items)

```
Total Cost := SUMX(order_items, RELATED(inventory_items[cost]))

Net Revenue := CALCULATE(SUM(order_items[sale_price]),
    order_items[status] <> "Cancelled", order_items[status] <> "Returned")

Gross Profit After Returns&Cancel := [Net Sales] - ([Total Cost of Sold Items] - ([Returned Cost] + [Cancelled Cost]))
```

## Returns

```
Cancelled Units := CALCULATE(COUNTROWS(order_items), order_items[status] = "Cancelled")

Delivered Units := CALCULATE(COUNTROWS(order_items),
    FILTER(order_items, NOT(ISBLANK(order_items[delivered_at]))))

Return Rate (Valid) := DIVIDE([Returned Units], [Total Units Sold] - [Cancelled Units])

Return Rate (Delivered) := DIVIDE([Returned Units], [Delivered Units])

Return Rate (300+) := IF([Delivered Units] >= 300, [Return Rate (Delivered)])
```

Why delivered units: an item that has not been delivered cannot be returned. Return rate was 10.03% on all items, 11.79% without cancelled items, and 28.56% on delivered items.

## Products

```
Distinct Products Sold := DISTINCTCOUNT(order_items[product_id])

Products with Net Sales := CALCULATE(DISTINCTCOUNT(order_items[product_id]),
    order_items[status] <> "Cancelled", order_items[status] <> "Returned")
```

## Forecast helper

```
Units Daily := [Total Units Sold] + 0
```

Adding 0 turns blank days into 0, so every date appears in the pivot (Forecast Sheet needs a continuous timeline).

## Calculated columns

```
DC Name := LOOKUPVALUE(distribution_centers[name], distribution_centers[id], inventory_items[product_distribution_center_id])

Category := LOOKUPVALUE(products[category], products[id], inventory_items[product_id])

Price Band := SWITCH(TRUE(), products[retail_price] < 20, "1) Under $20",
    products[retail_price] < 50, "2) $20-$50",
    products[retail_price] < 100, "3) $50-$100", "4) $100+")
```

Other calculated columns: `stock_status` (inventory_items), `is_returned` and `order_date` (order_items), `event_date` (events).
