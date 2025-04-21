# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db["Clientes"].aggregate([
  { $unwind: "$cuentas" },
  {
    $group: {
      _id: "$cuentas.tipo_cuenta",
      total: { $sum: "$cuentas.saldo" }
    }
  }
])
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db["Transacciones"].aggregate([
  {
    $lookup: {
      from: "Clientes",
      localField: "cliente_ref",
      foreignField: "_id",
      as: "Cliente"
    }
  },
  {
    $group: {
      _id: { cliente: "$Cliente.cedula", tipo: "$tipo_transaccion" },
      totalMonto: { $sum: "$monto" },
      cantidad: { $sum: 1 }
    }
  },
  {
    $group: {
      _id: "$_id.cliente",
      transacciones: {
        $push: {
          tipo: "$_id.tipo",
          totalMonto: "$totalMonto",
          cantidad: "$cantidad"
        }
      }
    }
  }, 
  {
    $project: {
      cedula: "$_id",
      transacciones: 1,
      _id: 0
    }
  }
])
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db["Clientes"].aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.tarjetas" },
  {
    $match: { "cuentas.tarjetas.tipo_tarjeta": "credito" }
  },
  {
    $group: {
      _id: "$cedula",
      nombre: { $first: "$nombre" },
      correo: { $first: "$correo" },
      direccion: { $first: "$direccion" },
      cantidadTDC: { $sum: 1 },
      tarjetasCredito: { $push: "$cuentas.tarjetas" }
    }
  },
  {
    $match: {
      cantidadTDC: { $gt: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      cedula: "$_id",
      nombre: 1,
      correo: 1,
      direccion: 1,
      cantidadTDC: 1,
      tarjetasCredito: 1
    }
  }
])
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db["Transacciones"].aggregate([
  {
    $match: { "tipo_transaccion": "deposito" }
  },
  {
    $group: {
      _id: {
        medioPago: "$detalles_deposito.medio_pago", mes: {
          $dateToString: { format: "%Y-%m", date: "$fecha" }
        }
      },
      totalMonto: { $sum: "$monto" },
      cantidad: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      medioPago: "$_id.medioPago",
      mes: "$_id.mes",
      totalMonto: 1,
      cantidad: 1
    }
  },
  { $sort: { mes: 1 } }
])
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db["Transacciones"].aggregate([
  {
    $match: { "tipo_transaccion": "retiro" }
  },
  {
    $group: {
      _id: {
        cuenta: "$num_cuenta", dia: {
          $dateToString: { format: "%Y-%m-%d", date: "$fecha" }
        }
      },
      totalMonto: { $sum: "$monto" },
      cantidad: { $sum: 1 }
    }
  },
  {
    $match: { "cantidad": { $gt : 3 },  "totalMonto": { $gt : 1000000 }}
  },
  {
    $project: {
      _id: 0,
      cuenta: "$_id.cuenta",
      fecha: "$_id.dia",
      totalMonto: 1,
      cantidad: 1
    }
  },
])
```