
# Offline policy evaluation
[![PyPI version](https://badge.fury.io/py/offline-evaluation.svg)](https://badge.fury.io/py/offline-evaluation) [![](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

Implementations and examples of common offline policy evaluation methods in Python.

## Installation
`pip install offline-evaluation`

## Usage
```
from ope.methods import doubly_robust
```

Get some historical logs generated by a previous policy:
```
df = pd.DataFrame([
	{"context": {"p_fraud": 0.08}, "action": "blocked", "action_prob": 0.90, "reward": 0},
	{"context": {"p_fraud": 0.03}, "action": "allowed", "action_prob": 0.90, "reward": 20},
	{"context": {"p_fraud": 0.02}, "action": "allowed", "action_prob": 0.90, "reward": 10},
	{"context": {"p_fraud": 0.01}, "action": "allowed", "action_prob": 0.90, "reward": 20},     
	{"context": {"p_fraud": 0.09}, "action": "allowed", "action_prob": 0.10, "reward": -20},
	{"context": {"p_fraud": 0.40}, "action": "allowed", "action_prob": 0.10, "reward": -10},
 ])
```
Define a function that computes `P(action | context)` under the new policy:
```
def action_probabilities(context):
    epsilon = 0.10
    if context["p_fraud"] > 0.10:
        return {"allowed": epsilon, "blocked": 1 - epsilon}    
    return {"allowed": 1 - epsilon, "blocked": epsilon}
```
Conduct the evaluation:
```
doubly_robust.evaluate(df, action_probabilities)
> {'expected_reward_logging_policy': 3.33, 'expected_reward_new_policy': -28.47}

doubly_robust.evaluate(logs_df, action_probabilities, num_bootstrap_samples=50)
> {'expected_reward_logging_policy': {'value': 3.33,
    'confidence_interval_0.95': [-8.583080401464436, 14.317080401464438]},
   'expected_reward_new_policy': {'value': 2.56,
    'confidence_interval_0.95': [-116.30282526565165, 64.00162526565165]}}
```
This means the new policy is significantly worse than the logging policy.  Instead of A/B testing this new policy online, it would be better to test some other policies offline first.

See examples for more detailed tutorials.

## Supported methods

- [x] Inverse propensity scoring
- [x] Direct method
- [x] Doubly robust ([paper](https://arxiv.org/abs/1503.02834))
