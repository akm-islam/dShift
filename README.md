# Dshift

## Validating the one-month-ahead return

The return in `returns` is dated at the end of the month it belongs to. To compare it with the panel's `ret_exc_lead1m`, shift the return date back by one month-end before merging. For example, the return dated February month-end is compared with the January panel row for the same `id`.

```python
nxt = returns.assign(
    eom=returns.eom - pd.offsets.MonthEnd(1)
).rename(columns={"ret_exc": "ret_next_from_returns"})

m = panel[["id", "eom", "ret_exc_lead1m"]].merge(
    nxt, on=["id", "eom"], how="left"
)
both = m.dropna()

print(
    f"pairs compared: {len(both):,} | max abs diff: "
    f"{(both.ret_exc_lead1m - both.ret_next_from_returns).abs().max():.2e}"
)
print(
    "panel rows with a target but no t+1 row in the universe returns: "
    f"{(m.ret_exc_lead1m.notna() & m.ret_next_from_returns.isna()).sum():,} "
    "(delistings / leaving the universe)"
)
print(
    f"panel rows without a target: {panel.ret_exc_lead1m.isna().sum():,} "
    f"({panel.ret_exc_lead1m.isna().mean():.2%})"
)
```

A maximum absolute difference near zero indicates that the panel's lead return agrees with the return in the following month. Rows with a target but no matching return may represent companies that delisted or otherwise left the returns universe. Rows without a target have no available one-month-ahead return in the panel.
# dShift
