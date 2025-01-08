# PostgreSQL Monitoring Stack with Docker

This project sets up a comprehensive monitoring solution for PostgreSQL using Docker containers. The stack includes PostgreSQL, Prometheus, Grafana, PostgreSQL Exporter, and AlertManager.

## Prerequisites

- Docker and Docker Compose
- psql client (for testing)
    - On Arch Linux: `sudo pacman -S postgresql`
    - On Ubuntu: `sudo apt install postgresql-client`

## Directory Structure

Before starting, create the necessary directories with proper permissions:

```bash
mkdir -p data/{postgres,prometheus,grafana,alertmanager}
sudo chown 472:472 data/grafana          # Grafana user
sudo chown 1001:1001 data/{prometheus,alertmanager}  # Prometheus/AlertManager user
sudo chown 999:999 data/postgres         # PostgreSQL user
```

## Configuration Files

Create prometheus.yml in the project root:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'postgres'
    static_configs:
      - targets: ['exporter:9187']
```

## Starting the Stack

1. Start the services:
```bash
docker-compose up -d
```

2. Access the web interfaces:
    - Grafana: http://localhost:3000 (admin/admin)
    - Prometheus: http://localhost:9090
    - AlertManager: http://localhost:9093

3. Configure Grafana:
    - Add Prometheus data source (URL: http://prometheus:9090)
    - Import dashboard ID 9628 for PostgreSQL monitoring

## Generating Test Data

Connect to the PostgreSQL database:

```bash
psql -h localhost -U postgres -d postgres
# Password: postgres
```

Create test tables:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Insert sample products:

```sql
INSERT INTO products (name, price) VALUES 
('Phone', 599.99),
('Laptop', 1299.99),
('Headphones', 99.99),
('Tablet', 499.99),
('Smartwatch', 299.99);
```

Create and run the order generation function:

```sql
CREATE OR REPLACE FUNCTION generate_orders() RETURNS void AS $$
DECLARE
    product_count INTEGER;
BEGIN
    FOR i IN 1..100 LOOP
        PERFORM pg_sleep(random() * 0.1);
        INSERT INTO orders (product_id, quantity)
        VALUES (
            floor(random() * 5 + 1)::int,
            floor(random() * 5 + 1)::int
        );
    END LOOP;
END;
$$ LANGUAGE plpgsql;

SELECT generate_orders();
```

## Troubleshooting

If you encounter permission issues:
1. Ensure proper directory ownership as described above
2. Check container logs: `docker-compose logs service_name`
3. If problems persist, remove data directories and recreate them with correct permissions

To cleanly restart the stack:
```bash
docker-compose down
sudo rm -rf data/*  # Be careful with this command
# Recreate directories with proper permissions
docker-compose up -d
```

## Container Ports

- PostgreSQL: 5432
- Prometheus: 9090
- Grafana: 3000
- AlertManager: 9093
- Postgres Exporter: 9187