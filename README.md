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
## Parte 2 — Refactor DIP, integración y CI/CD

### Diagnóstico: por qué `enviarRecordatorioUrgente` no se puede testear

La primera versión de `enviarRecordatorioUrgente` construye el canal con
`new SmsReminderChannel(new TwilioNotificadorSmsGateway())` (línea 54 de
`CitaReminderServiceImpl`). Como el objeto se crea dentro del cuerpo del método,
la clase decide por sí misma qué implementación concreta usa y no existe ningún
punto por donde una prueba pueda sustituirlo por un test double. Por eso no se
puede escribir la prueba "el gateway de SMS falla y la cita no debe marcarse como
enviada".

En contraste, `enviarRecordatoriosPendientes()` sí es testeable desde la Parte 1:
recibe el repositorio, la Factory y el `Clock` por constructor, así que la prueba
puede inyectarle mocks y un reloj fijo.

Técnicamente se podría interceptar ese `new` con `Mockito.mockConstruction()`,
pero eso es una señal de que el problema es el diseño y no la herramienta: se
resuelve aplicando DIP, no con una API de pruebas más avanzada.
### Pruebas de integración: por qué H2 real pero gateway mockeado

`CitaRepositoryTest` (`@DataJpaTest`) prueba la consulta derivada del repositorio
contra una H2 en memoria, y `CitaReminderServiceIntegrationTest`
(`@SpringBootTest`) levanta el contexto completo de Spring con un `Clock` fijo y
un `@MockBean` sobre `NotificadorSmsGateway`.

No es inconsistente que convivan una base de datos real y un mock. H2 es un fake
aceptable porque su contrato SQL/JPA es equivalente al de la base real y no
requiere infraestructura adicional. Un proveedor de SMS de terceros no tiene un
equivalente gratuito y determinista; mockearlo evita depender de credenciales,
de la red y de la disponibilidad del proveedor en cada ejecución del pipeline.

Observación: en Spring Boot 3.5 `@MockBean` está marcado como deprecated y su
reemplazo es `@MockitoBean`. Se usó `@MockBean` por seguir el enunciado.