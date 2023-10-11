## Forta Toolkit

Various tools to help with the common problems of Forta bot development.

## Installation

```bash
# globally
pip install forta_toolkit

# in a local environment
poetry add forta_toolkit
```

## Usage

### Bot setup

The Forta often require initialization steps to adapt to a given chain or use external tools.

### Alert statistics

This is an alternative to querying the Zetta API for alert statistics.
It saves a local history of the alerts in memory and use it to calculate the rates.
The main motivation is to improve performance by avoiding web requests.

To use it, just wrap `handle_block` / `handle_transaction` / `handle_alert` as follows:

```python
import forta_toolkit

@forta_toolkit.alerts.alert_history(size=10000)
def handle_block(log: BlockEvent) -> list:
    pass

@forta_toolkit.alerts.alert_history(size=10000)
def handle_transaction(log: TransactionEvent) -> list:
    pass

@forta_toolkit.alerts.alert_history(size=10000)
def handle_alert(log: AlertEvent) -> list:
    pass
```

The decorator will automatically add the `anomaly_score` in the metadata of the `Finding` objects.
It will use the field `alert_id` from the `Finding` objects to identify them.

> make sure the history size is big enough to contain occurences of the bot alerts!

For example, if your bot triggers `ALERT-1` every 2k transactions and `ALERT-2` every 10k on average:
`@alert_history(size=100000)` would gather enough alerts to have a relevant estimation of the rate of both alerts.

### Improving performances

### Logging execution events

### Profiling

The bots have to follow the pace of the blockchain, so they need to process transactions relatively quickly.

You can leverage the profiling tools to find the performance bottlenecks in your bots:

```python
from forta_toolkit.profiling import test_performances, display_performances

test_performances(func=handle_transaction, data=some_tx_log)
display_performances(logpath='./test_performances')
```

Otherwise, you can monitor the performances directly when processing mainnet transactions.
Just decorate the `handle_block` / `handle_transaction` / `handle_alert` as follows:

```python
@forta_toolkit.alerts.profile
def handle_transaction(tx: TransactionEvent) -> list:
    pass
```

Then you can parse the profile logs manually with `pstats` or:

```python
display_performances(logpath='some/path/to/the/logs/handle_transaction')
```