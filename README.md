# MongoDB Backend for MLflow Registered Models

> [!WARNING]
> This project is under active development and has not reached a stable release. APIs, storage
> formats, and configuration may change at any time without notice. **It is not intended for
> production use.** See [LICENSE](LICENSE) for the full disclaimer of warranties.

An MLflow plugin that stores registered models, versions, aliases, tags, and related metadata
in MongoDB. MLflow tracking data, such as experiments and runs, remains in a separate backend
store.

## Installation

Python 3.10 or newer, MLflow 3.1 or newer, and a MongoDB deployment are required.
For a quick start, install the plugin directly from GitHub into the same Python environment
that will run the MLflow server. If you use SSH, make sure your SSH key is configured with
GitHub:

```bash
python -m pip install "git+ssh://git@github.com/mongodb-developer/mlflow-mongodb.git"
```

If SSH is not configured, use HTTPS instead:

```bash
python -m pip install "git+https://github.com/mongodb-developer/mlflow-mongodb.git"
```

Registry URIs using either `mongodb://` or `mongodb+srv://` are routed to this backend.

## Running MLflow

The tracking store and registered-model store are configured independently. For example, the
following configuration keeps experiments and runs in SQLite and stores registered models
and their versions in MongoDB:

```bash
export TRACKING_STORE_URI="sqlite:///tracking.db"
export REGISTRY_STORE_URI="mongodb://localhost:27017/mlflow_registry"
export MLFLOW_BIND_HOST="127.0.0.1"
export MLFLOW_BIND_PORT="5000"

mlflow server \
  --backend-store-uri "${TRACKING_STORE_URI}" \
  --registry-store-uri "${REGISTRY_STORE_URI}" \
  --host "${MLFLOW_BIND_HOST}" \
  --port "${MLFLOW_BIND_PORT}"
```

Open `http://127.0.0.1:5000` to use the MLflow UI. Clients can connect to the server with:

```bash
export MLFLOW_TRACKING_URI="http://127.0.0.1:5000"
```

The MongoDB URI must include the database name; `mlflow_registry` is the database name in
the example above. Authentication, replica-set, TLS, and other connection options can be
provided using standard MongoDB URI syntax. For example:

```text
mongodb://username:password@mongo.example.com:27017/mlflow_registry?authSource=admin
```

## Documentation

New to MLflow? Start with the [MLflow Tracking Quickstart](https://mlflow.org/docs/latest/ml/getting-started/quickstart/).
The following guides provide additional context for using this plugin:

- [Model Registry Tutorial](https://mlflow.org/docs/latest/ml/model-registry/tutorial) — register and manage model versions.
- [Backend Stores](https://mlflow.org/docs/latest/self-hosting/architecture/backend-store/) — understand where MLflow stores tracking data.
- [Tracking Server Configuration](https://mlflow.org/docs/latest/self-hosting/architecture/tracking-server/) — configure tracking and registry stores independently.
- [MongoDB Connection Strings](https://www.mongodb.com/docs/manual/reference/connection-string/) — configure authentication, TLS, replica sets, and other URI options.

## Versioning

This project follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`), using the
same release-versioning approach as
[mongo-hibernate](https://github.com/mongodb/mongo-hibernate), adapted to Python's
[PEP 440](https://peps.python.org/pep-0440/) version identifiers:

- **`main` / in-development version** — carries a `.devN` suffix (the PEP 440 equivalent of
  Maven's `-SNAPSHOT`), e.g. `0.1.0.dev0`. This means "work in progress toward `0.1.0`, not yet
  released."
- **Pre-releases** — cut from `main` as `<base-version>a<N>` (alpha) or `<base-version>b<N>`
  (beta), with `N` starting at `0` and incrementing for each subsequent pre-release of that base
  version, e.g. `0.1.0a0`, `0.1.0a1`, `0.1.0b0`. Per MongoDB's customer-facing
  [release stage terminology](https://mongodb-mops.zendesk.com/hc/en-us/articles/43770296682259-Release-Stage-Terminology),
  these correspond to **Private Preview** (alpha) and **Public Preview** (beta) — "alpha"/"beta"
  remain the internal/engineering version identifiers; use "Private Preview"/"Public Preview" in
  any customer-facing communication.
- **Stable releases** — drop the pre-release suffix entirely, e.g. `0.1.0`. This is the default,
  General Availability (GA) state; a version does not need to be labeled "GA."
- After a stable release ships, `main` moves to the next `.devN` version (e.g. `0.2.0.dev0`
  following `0.1.0`).

The package remains on major version `0` (`0.y.z`) until the public API is considered stable;
per semver, anything in `0.y.z` may change at any time.

## Contributor setup

For development, clone the repository and install it in editable mode:

```bash
git clone https://github.com/mongodb-developer/mlflow-mongodb.git
cd mlflow-mongodb

python -m venv .venv
source .venv/bin/activate
python -m pip install -e .
```

If you downloaded a source archive instead, extract it and run the `venv` and `pip` commands
from the extracted repository root. The final `.` in `pip install -e .` means “install the
project in the current directory,” and `-e` keeps the installation linked to that source
checkout.

Install the development dependencies and Git hooks with:

```bash
python -m pip install -e ".[dev]"
pre-commit install --install-hooks
```

Run all formatting, linting, and repository checks with:

```bash
pre-commit run --all-files
```

## Functional tests

The functional tests use a real MongoDB 8.0 or newer server. Install the project in editable
mode with its development dependencies before running them:

```bash
python -m pip install -e ".[dev]"
```

Set `MONGODB_URI` to a dedicated test database. The database name must contain `test` as a
distinct hyphen- or underscore-separated segment; for example:

```bash
export MONGODB_URI="mongodb://localhost:27017/mlflow_functional_test"
python -m pytest tests/functional
```

Authentication, replica-set, TLS, and other standard MongoDB URI options can be included in
`MONGODB_URI`. Use a replica set when running tests that exercise MongoDB transactions.

The functional suite deletes documents from its application collections after every test so
their indexes can be reused. At the end of the test session, it drops the database selected by
`MONGODB_URI`. Never point `MONGODB_URI` at a database containing data that must be preserved.
