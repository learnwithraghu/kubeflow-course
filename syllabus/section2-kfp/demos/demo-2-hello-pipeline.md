# Demo 2.1: Hello World Pipeline

## Overview
Build and run a simple "Hello World" pipeline to understand KFP basics.

## Time
15 minutes

## Steps

### 1. Create Directory Structure
```bash
mkdir -p ~/kubeflow-demo/components/hello
cd ~/kubeflow-demo
```

### 2. Create Hello Component

**components/hello/component.yaml**:
```yaml
name: Hello World
description: Simple hello world component
inputs:
  - {name: name, type: String, description: 'Name to greet'}
outputs:
  - {name: greeting, type: String}
implementation:
  container:
    image: python:3.9
    command:
      - echo
    args:
      - "Hello, {inputValue: name}!"
      - {outputPath: greeting}
```

### 3. Create Pipeline

**pipeline.py**:
```python
import kfp
from kfp import dsl
from kfp.components import load_component_from_file

@dsl.pipeline(name='hello-world', description='Simple hello world')
def hello_pipeline(name: str = "Student"):
    hello = load_component_from_file('components/hello/component.yaml')(name=name)
```

### 4. Compile and Upload
```bash
# Compile
kfp compiler pipeline.py --output hello.yaml

# Upload via UI (Pipelines → Upload)
# Or use SDK:
# client = kfp.Client()
# client.upload_pipeline('hello.yaml', name='hello-world')
```

### 5. Run Pipeline
- Create run with name="World"
- Watch graph view
- Check output logs

## Expected Output
```
Hello, World!
```

## Key Learning Points
- Components are independent containers
- Pipeline connects components via inputs/outputs
- KFP handles orchestration and monitoring

---

## Demo 2.2: Parameterized Pipeline

## Overview
Extend hello pipeline to use parameters.

## Steps

### Modified Pipeline
```python
@dsl.pipeline(name='greeting-pipeline')
def greet_pipeline(
    first_name: str = "Hello",
    last_name: str = "World",
    exclamation: bool = True
):
    hello = hello_op(name=f"{first_name} {last_name}!")
```

## Try It
1. Change first_name to "Kubeflow"
2. Set exclamation=False
3. Compare runs
