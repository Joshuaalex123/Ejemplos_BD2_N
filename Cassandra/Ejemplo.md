# Cassandra 

## Caso de uso: Métricas de sensores IoT (Serie de tiempo)

Cassandra brilla cuando hay **escrituras masivas** y consultas por rango de tiempo.
Es una base de datos **orientada a columnas**, distribuida y sin nodo maestro.

---

## 1. Levantar el contenedor

```bash
docker-compose up -d
```





## 4. Explorar los datos

### Ver los keyspaces disponibles
```cql
DESCRIBE KEYSPACES;
```

### Cambiar al keyspace
```cql
USE iot;
```

### Ver las tablas
```cql
DESCRIBE TABLES;
```

### Consultar todas las lecturas de un sensor
```cql
SELECT * FROM sensores WHERE dispositivo_id = 'sensor-01';
```

### Consultar por rango de tiempo
```cql
SELECT * FROM sensores
WHERE dispositivo_id = 'sensor-01'
  AND timestamp >= '2026-03-01 00:00:00'
  AND timestamp <= '2026-03-01 23:59:59';
```

### Consultar solo temperatura alta
```cql
SELECT dispositivo_id, timestamp, temperatura
FROM sensores
WHERE dispositivo_id = 'sensor-02'
  AND temperatura > 30
ALLOW FILTERING;
```


## 6. Demostración de escalabilidad

```cql
-- Insertar 5 lecturas masivas de golpe
BEGIN BATCH
  INSERT INTO iot.sensores (dispositivo_id, timestamp, temperatura, humedad, ubicacion) VALUES ('sensor-01', '2026-03-01 10:00:00', 23.1, 55.0, 'Planta Baja');
  INSERT INTO iot.sensores (dispositivo_id, timestamp, temperatura, humedad, ubicacion) VALUES ('sensor-01', '2026-03-01 10:01:00', 23.4, 54.8, 'Planta Baja');
  INSERT INTO iot.sensores (dispositivo_id, timestamp, temperatura, humedad, ubicacion) VALUES ('sensor-01', '2026-03-01 10:02:00', 23.8, 54.5, 'Planta Baja');
  INSERT INTO iot.sensores (dispositivo_id, timestamp, temperatura, humedad, ubicacion) VALUES ('sensor-02', '2026-03-01 10:00:00', 31.2, 40.1, 'Azotea');
  INSERT INTO iot.sensores (dispositivo_id, timestamp, temperatura, humedad, ubicacion) VALUES ('sensor-03', '2026-03-01 10:00:00', 19.9, 70.3, 'Sótano');
APPLY BATCH;
```

---

## 7. Comparación con SQL

```sql
-- En SQL necesitarías:
CREATE TABLE sensores (
  id INT PRIMARY KEY AUTO_INCREMENT,
  dispositivo_id VARCHAR(50),
  timestamp DATETIME,
  temperatura FLOAT,
  INDEX idx_dispositivo (dispositivo_id),
  INDEX idx_timestamp (timestamp)
);
-- Los índices se degradan con millones de filas
-- Escalar horizontalmente requiere sharding manual
```

```cql
-- En Cassandra:
-- La Primary Key ya define partición y orden
-- Distribuido por diseño
-- Escritura O(1) sin locks
```


