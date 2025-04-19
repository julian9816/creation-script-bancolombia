# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT *
FROM (
    SELECT
        cliente.id_cliente,
        COUNT(cuenta.num_cuenta) AS cantidadCuentas,
        SUM(cuenta.saldo) AS totalSaldo
    FROM
        cuenta
    INNER JOIN cliente ON cliente.id_cliente = cuenta.id_cliente
    GROUP BY cliente.id_cliente
) AS sub
WHERE sub.cantidadCuentas > 1
ORDER BY sub.totalSaldo;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT
    cliente.id_cliente,
    transaccion.tipo_transaccion,
    SUM(transaccion.monto) AS total_monto
FROM
    cuenta
INNER JOIN cliente ON cliente.id_cliente = cuenta.id_cliente
INNER JOIN transaccion ON transaccion.num_cuenta = cuenta.num_cuenta
GROUP BY
    cliente.id_cliente,
    transaccion.tipo_transaccion

EXCEPT

SELECT
    cliente.id_cliente,
    transaccion.tipo_transaccion,
    SUM(transaccion.monto)
FROM
    cuenta
INNER JOIN cliente ON cliente.id_cliente = cuenta.id_cliente
INNER JOIN transaccion ON transaccion.num_cuenta = cuenta.num_cuenta
INNER JOIN transferencia ON transferencia.id_transaccion = transaccion.id_transaccion
GROUP BY
    cliente.id_cliente,
    transaccion.tipo_transaccion

ORDER BY id_cliente;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT
    cuenta.num_cuenta
FROM
    cuenta
LEFT JOIN tarjeta ON tarjeta.num_cuenta = cuenta.num_cuenta
WHERE tarjeta.id_tarjeta IS NULL;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT
    cuenta.tipo_cuenta,
    AVG(cuenta.saldo) AS promedio_saldo
FROM
    cuenta
INNER JOIN cuentaahorro ON cuenta.num_cuenta = cuentaahorro.num_cuenta
WHERE EXISTS (
    SELECT 1
    FROM transaccion
    WHERE transaccion.num_cuenta = cuenta.num_cuenta
      AND transaccion.fecha >= CURRENT_DATE - INTERVAL '30 days'
)
GROUP BY cuenta.tipo_cuenta

UNION

SELECT
    cuenta.tipo_cuenta,
    AVG(cuenta.saldo) AS promedio_saldo
FROM
    cuenta
INNER JOIN cuentacorriente ON cuenta.num_cuenta = cuentacorriente.num_cuenta
WHERE EXISTS (
    SELECT 1
    FROM transaccion
    WHERE transaccion.num_cuenta = cuenta.num_cuenta
      AND transaccion.fecha >= CURRENT_DATE - INTERVAL '30 days'
)
GROUP BY cuenta.tipo_cuenta;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT
    cuenta.id_cliente
FROM
    cuenta
INNER JOIN transaccion ON transaccion.num_cuenta = cuenta.num_cuenta
WHERE transaccion.descripcion != 'Retiro en cajero automático'
AND EXISTS (
    SELECT 1 FROM transaccion
    INNER JOIN transferencia ON transferencia.id_transaccion = transaccion.id_transaccion
    WHERE transaccion.num_cuenta = cuenta.num_cuenta
)
GROUP BY cuenta.id_cliente;
```