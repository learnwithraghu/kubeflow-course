# Exercise 2: Build a 3-Step Pipeline

## Objective
Build a simple pipeline that downloads, preprocesses, and trains a model.

## Pipeline Structure

```
download-data → preprocess-data → train-model
```

## Tasks

### Task 1: Create Project Structure
```bash
mkdir -p my-pipeline/{components,pipelines}
cd my-pipeline
```

### Task 2: Create Download Component

**components/download/component.yaml**:
```yaml
name: Download Dataset
description: Download MNIST dataset
outputs:
  - {name: data, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        echo "Downloading MNIST dataset..."
        python3 -c "
        import urllib.request
        # Simulated download
        with open('/tmp/mnist_data.csv', 'w') as f:
            f.write('pixel1,pixel2,label\n')
            for i in range(100):
                f.write(','.join(['0']*784) + ',0\n')
        "
        cat /tmp/mnist_data.csv
    args:
      - {outputPath: data}
```

### Task 3: Create Preprocess Component

**components/preprocess/component.yaml**:
```yaml
name: Preprocess Data
description: Normalize dataset
inputs:
  - {name: data, type: Dataset}
outputs:
  - {name: processed_data, type: Dataset}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        echo "Preprocessing data..."
        python3 -c "
        with open('$(inputs.data)', 'r') as f:
            lines = f.readlines()
        # Normalize (just header + first row for demo)
        with open('/tmp/processed.csv', 'w') as f:
            f.write(lines[0])  # keep header
            if len(lines) > 1:
                f.write(lines[1])  # one row
        "
        echo "Preprocessing complete"
    args:
      - {inputPath: data}
      - {outputPath: processed_data}
```

### Task 4: Create Train Component

**components/train/component.yaml**:
```yaml
name: Train Model
description: Train simple ML model
inputs:
  - {name: data, type: Dataset}
parameters:
  - {name: epochs, type: Integer, default: '5'}
outputs:
  - {name: model, type: Model}
implementation:
  container:
    image: python:3.9
    command:
      - sh
      - -c
      - |
        echo "Training for $0 epochs..."
        python3 -c "
        print('Simulating training...')
        with open('/tmp/model.pt', 'w') as f:
            f.write('model v1.0')
        print('Training complete')
        "
        echo "Model saved"
    args:
      - {inputValue: epochs}
      - {inputPath: data}
      - {outputPath: model}
```

### Task 5: Create Pipeline Definition

**pipelines/pipeline.py**:
```python
import kfp
from kfp import dsl
from kfp.components import load_component_from_file

@kfp.dsl.pipeline(name='simple-ml-pipeline', description='Download, preprocess, train')
def ml_pipeline(url: str = "http://example.com/data", epochs: int = 5):
    download = load_component_from_file('../components/download/component.yaml')(url=url)
    preprocess = load_component_from_file('../components/preprocess/component.yaml')(data=download.output)
    train = load_component_from_file('../components/train/component.yaml')(data=preprocess.output, epochs=epochs)
```

### Task 6: Compile Pipeline

```bash
# Install KFP SDK if not already
pip install kfp

# Compile
kfp_COMPILER=dsl-compile python -m kfp.compiler pipeline.py --output pipeline.yaml
```

### Task 7: Upload and Run

1. Open Kubeflow Dashboard: http://localhost:8080
2. Go to Pipelines → Upload pipeline
3. Upload `pipeline.yaml`
4. Create run with default parameters
5. Monitor execution in Graph view

## Success Criteria

- [ ] Pipeline compiles without errors
- [ ] Pipeline visible in Kubeflow UI
- [ ] Run completes with all 3 components
- [ ] Can see outputs in run details

## Cleanup

```bash
# Delete pipeline from UI or via kubectl
kubectl delete -f pipeline.yaml -n kubeflow
```
