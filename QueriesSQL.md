# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT  A.id_cliente, A.cedula, A.nombre, COUNT(B.num_cuenta) AS cantidad_cuentas, SUM(B.saldo) AS saldo_total
FROM cliente A JOIN cuenta B ON A.id_cliente = B.id_cliente
GROUP BY A.id_cliente, A.cedula, A.nombre
HAVING COUNT(B.num_cuenta) > 1
ORDER BY saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT
    A.id_cliente,
    A.cedula,
    A.nombre, 
    SUM(CASE WHEN C.tipo_transaccion = 'deposito' THEN C.monto ELSE 0 END) AS total_depositos,
    SUM(CASE WHEN C.tipo_transaccion = 'retiro' THEN C.monto ELSE 0 END) AS total_retiros
FROM
    cliente A 
    JOIN cuenta B ON A.id_cliente = B.id_cliente
    JOIN transaccion C ON B.num_cuenta = C.num_cuenta
GROUP BY A.id_cliente, A.cedula, A.nombre
ORDER BY A.id_cliente;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT A.id_cliente, C.nombre, A.num_cuenta
FROM
    cuenta A
    JOIN cliente C ON A.id_cliente = C.id_cliente
    LEFT JOIN tarjeta B ON A.num_cuenta = B.num_cuenta
WHERE B.num_cuenta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT tipo_cuenta, AVG(saldo) AS saldo_promedio
FROM cuenta
WHERE num_cuenta IN (
    SELECT num_cuenta
    FROM transaccion
    WHERE fecha >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY num_cuenta
)
GROUP BY tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT A.id_cliente, A.cedula, A.nombre
FROM cliente A JOIN cuenta B ON A.id_cliente = B.id_cliente
WHERE B.num_cuenta IN (
    SELECT C.num_cuenta
    FROM transaccion C
    WHERE C.tipo_transaccion = 'transferencia'
    GROUP BY C.num_cuenta
)
AND B.num_cuenta NOT IN (
    SELECT C.num_cuenta
    FROM transaccion C JOIN retiro D ON C.id_transaccion = D.id_transaccion
    WHERE D.canal = 'cajero'
    GROUP BY C.num_cuenta
)
GROUP BY A.id_cliente, A.cedula, A.nombre;
```
