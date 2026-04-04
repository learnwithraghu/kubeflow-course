# Exercise 3: Artifact Passing

## Objective
Learn to pass data artifacts between pipeline components.

## Tasks

### Task 1: Create Components for Data Passing

**components/save_number/component.yaml**:
```yaml
name: Save Number
description: Save a number to file
outputs:
  - {name: number_file, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        python3 -c "
        import json
        data = {'number': 42, 'name': 'answer'}
        with open('/tmp/number.json', 'w') as f:
            json.dump(data, f)
        "
    args:
      - {outputPath: number_file}
```

**components/double_number/component.yaml**:
```yaml
name: Double Number
description: Read number and double it
inputs:
  - {name: number_file, type: Dataset}
outputs:
  - {name: result_file, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        python3 -c "
        import json
        with open('$(inputs.number_file)', 'r') as f:
            data = json.load(f)
        data['doubled'] = data['number'] * 2
        with open('/tmp/result.json', 'w') as f:
            json.dump(data, f)
        print('Result:', data)
        "
    args:
      - {inputPath: number_file}
      - {outputPath: result_file}
```

**components/display/component.yaml**:
```yaml
name: Display Result
description: Display the result
inputs:
  - {name: result_file, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        cat $(inputs.result_file)
    args:
      - {inputPath: result_file}
```

### Task 2: Create Pipeline

**pipelines/pipeline.py**:
```python
import kfp
from kfp import dsl
from kfp.components import load_component_from_file

@dsl.pipeline(name='artifact-pipeline', description='Pass artifacts between components')
def artifact_pipeline():
    save = load_component_from_file('../components/save_number/component.yaml')()
    doubled = load_component_from_file('../components/double_number/component.yaml')(number_file=save.output)
    display = load_component_from_file('../components/display/component.yaml')(result_file=doubled.output)
```

### Task 3: Compile and Run

```bash
kfp compiler pipelines/pipeline.py --output pipeline.yaml
```

Upload to Kubeflow and run.

## Expected Output
```json
{"number": 42, "name": "answer", "doubled": 84}
```

## Success Criteria
- [ ] Pipeline runs successfully
- [ ] Each component executes in order
- [ ] Data flows between components correctly
- [ ] Final output shows doubled value

## Challenge Exercise
Add a component that multiplies by 3 instead of 2. Can you make it conditional?

## Cleanup
```bash
kubectl delete -f pipeline.yaml -n kubeflow
```
