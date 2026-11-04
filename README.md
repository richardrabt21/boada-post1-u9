# Recordatorios Citas Service

## Descripción

Repositorio del post-contenido de la Unidad 9 de Patrones de Diseño de Software.
Un único proyecto Spring Boot que envía recordatorios de citas por el canal
preferido del paciente (correo o SMS), construido desde el inicio con Strategy
(`ReminderChannel`) y Factory Method (`ReminderChannelFactory`) inyectados por
constructor.

## Parte 1 — Pruebas unitarias con JUnit 5 y Mockito

`CitaReminderServiceImplTest` aísla `CitaReminderServiceImpl` de sus tres
colaboradores. Ver "Decisiones de diseño" para la justificación de por qué cada
uno se mockeó o se resolvió con un fake.

## Decisiones de diseño

### Mock, stub o fake para cada colaborador (Parte 1)

- **`CitaRepository` → Mock.** No basta con stubbear la consulta para que
  devuelva una lista: también hay que verificar que `save()` se llama una vez y
  con la cita marcada como enviada. Cuando una prueba necesita controlar la
  respuesta y además verificar la interacción, es un mock.
- **`ReminderChannelFactory` / `ReminderChannel` → Mock.** El comportamiento bajo
  prueba es la selección de estrategia: que el servicio pida el canal correcto
  y que ese canal reciba el mensaje. Eso solo se comprueba verificando la
  interacción, no con un valor de retorno.
- **`Clock` → Fake (no mock).** `Clock.fixed(...)` es una implementación real y
  completa que provee el JDK y se comporta igual que en producción, solo con el
  instante congelado. Mockearlo exigiría stubbear `instant()` y `getZone()` por
  separado y sería menos fiel al comportamiento real.

## Ejecutar las pruebas

```bash
mvn test
```

## Herramientas utilizadas

- Java 21, Spring Boot 3.5, Spring Data JPA, H2, JUnit 5, Mockito
- Apache Maven, Git, GitHub