# Graph-repairing-under-neighborhood-constraints

## Quickstart: Environment Setup & Neo4j Database Creation

### 1. Configure Environment Variables

Create a `.env` file in the project root 
Use `env.txt` as a template.

**Example contents:**
```
- `NEO4J_URI`: Bolt URI for your Neo4j instance (default for local Neo4j Desktop)
- `NEO4J_USERNAME` / `NEO4J_PASSWORD`: Your Neo4j credentials
- `NEO4J_CONSTRAINT_DB` / `NEO4J_INSTANCE_DB`: Names for your two databases
- `FRAUD_NUMBER`: Number of swaps/perturbations to perform (for experiments)
```

### 2. Install Dependencies

Run the following command in your project directory:
```bash
uv sync
```

This installs all required Python packages from `pyproject.toml`.

### 3. Create Neo4j Databases

Open Neo4j Desktop or Neo4j Browser and run:
```cypher
CREATE DATABASE constraint_db;
CREATE DATABASE instance_db;
```
- Use simple, lowercase names (no spaces or special characters).
- You can change these names in your .env file.

Tip:
Never commit your .env file (but it's gitignore so you shouldn't be able to)
