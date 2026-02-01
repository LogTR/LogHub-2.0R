# LogHub-2.0R: Refined Log Parsing Benchmark Dataset

This repository contains **LogHub-2.0R**, a refined version of the LogHub-2.0 benchmark dataset for log parsing evaluation. The refinements were produced by [LogTR](https://github.com/LogTR/LogTR), an automated framework for log template auditing and repair.

## Overview

LogHub-2.0 is widely regarded as the most authoritative benchmark for evaluating log parsing techniques. However, our research discovered substantial **structural label noise** in the original dataset, including:

- **Missing Structural Information**: Templates incorrectly identify fixed syntactic symbols (such as specific prefixes or connectors) as parameters. This causes extracted parameters to contain structures that should belong to the template, introducing noise.
- **Missing Parameters**: Parameter structures are over-simplified. This deviation in parsing granularity causes critical numerical distribution information to be lost during extraction, severely affecting downstream modeling.
- **Template Over-Merging**: Different log templates are incorrectly merged into one. This noise causes extracted log templates to be distorted, compromising the accuracy of generated logs.

LogHub-2.0R addresses these issues by providing corrected templates verified through generative verification and confirmed by human experts.

## Key Statistics

| Metric | Value |
|--------|-------|
| Total templates refined | **493** |
| New templates from splitting | **20** |
| Percentage of templates refined | **14.7%** |
| Systems covered | **13** |

### Refinements by System

| System | Refined Templates |
|--------|-------------------|
| Thunderbird | 205 |
| Mac | 101 |
| Linux | 81 |
| Hadoop | 32 |
| Spark | 19 |
| BGL | 16 |
| HealthApp | 14 |
| Zookeeper | 9 |
| HPC | 5 |
| Apache | 4 |
| OpenStack | 3 |
| HDFS | 2 |
| OpenSSH | 2 |

## Files

### `LogHub2R.json`

Contains all template modifications in JSON format:

```json
{
  "statistics": {
    "total_modifications": 493,
    "by_dataset": { ... },
    "datasets_count": 13
  },
  "modifications": {
    "Apache": [
      {
        "event_id": "E10",
        "original": "env.createBean2(): Factory error creating <*> (<*>, <*>)",
        "modified": "env.createBean2(): Factory error creating <*> ( <*>, <*>)"
      },
      ...
    ],
    ...
  }
}
```

Each modification entry contains:
- `event_id`: The EventId in the original LogHub-2.0 dataset
- `original`: The original template from LogHub-2.0
- `modified`: The corrected template in LogHub-2.0R

### `split_temp.txt`

Contains templates that were split from over-merged entries. When a single template in LogHub-2.0 incorrectly covered multiple distinct log types, LogTR identified and separated them into multiple precise templates.

Format:
```
[System Name]

[Original EventId],[Original Template]
[Original EventId],[Split Template 1]
[Original EventId],[Split Template 2]
...
```

Example (BGL E284):
```
BGL

E284,Target=<*> Message=<*> JtagId = <*>
E284,Target=<*> Message=<*> JtagId = <*>
E284,Target=<*> Message=<*> JtagId = <*> Run environmental monitor to diagnose possible hardware failure.
```

This indicates that the original E284 template was over-merged and should be split into two distinct templates.

## Usage

### Applying Refinements

To use LogHub-2.0R, replace the original templates in LogHub-2.0 with the modified versions:

```python
import json

# Load refinements
with open('LogHub2R.json', 'r') as f:
    refinements = json.load(f)

# Apply to your dataset
for system, modifications in refinements['modifications'].items():
    for mod in modifications:
        event_id = mod['event_id']
        original = mod['original']
        modified = mod['modified']
        # Replace original template with modified version
        # in your LogHub-2.0 dataset
```

### Handling Split Templates

For split templates in `split_temp.txt`, you need to:
1. Create new EventIds for the split templates (e.g., E284 → E284, E284X)
2. Re-assign logs to the appropriate new templates based on pattern matching

## Types of Refinements

### 1. Missing Structural Information
Templates incorrectly identify fixed syntactic symbols (such as specific prefixes or connectors) as parameters. This causes extracted parameters to contain structures that should belong to the template, introducing noise.
```
Original (E94): "<*> ddr errors(s) detected and corrected on rank <*>, symbol <*> bit <*>"
Modified (E94): "<*> ddr errors(s) detected and corrected on rank <*>, symbol <*>, bit <*>"
```

### 2. Missing Parameters
Parameter structures are over-simplified. This deviation in parsing granularity causes critical numerical distribution information to be lost during extraction, severely affecting downstream modeling.
```
Original (E214): "correctable error detected in directory <*>"
Modified (E214): "correctable error detected in directory <*>......<*>"
```

### 3. Template Over-Merging
Different log templates are incorrectly merged into one. This noise causes extracted log templates to be distorted, compromising the accuracy of generated logs. (see `split_temp.txt`)
```
Original (E229): "invalid (<*>)........................<*>"
Split into:
  - (E229): "invalid (<*>)........................<*>"
  - (E229): "invalid (<*>)...........................<*>"
```



## Related Resources

- [LogTR Source Code](https://github.com/LogTR/LogTR) - The automated framework used to generate these refinements
- [LogHub-2.0](https://github.com/logpai/loghub-2.0) - The original benchmark dataset



## Acknowledgments

We thank the [LOGPAI team](https://github.com/logpai) for their foundational work in creating the original benchmark dataset for log parsing evaluation.




