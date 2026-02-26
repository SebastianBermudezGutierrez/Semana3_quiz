## Fase 1 - Levantar el ambiente

## Fase 2 — Tabla de Hallazgos

| # | Descripción del problema | Archivo | Línea aprox. | Principio violado | Riesgo |
|---|--------------------------|---------|--------------|------------------|--------|
| 1 | Uso de Statement con concatenación SQL (SQL Injection en login) | UserRepository.java | ~20 | Seguridad – SQL Injection | Alto |
| 2 | Uso de Statement en INSERT (SQL Injection en registro) | UserRepository.java | ~34 | Seguridad – SQL Injection | Alto |
| 3 | Credenciales de base de datos hardcodeadas | UserRepository.java | ~11-13 | Seguridad – Secretos en código | Alto |
| 4 | Uso de MD5 para hashing de contraseñas | AuthService.java | ~60 | Seguridad – Hash débil | Alto |
| 5 | Se retorna el hash de la contraseña en login | AuthService.java | ~28 | Exposición de datos sensibles | Alto |
| 6 | Password y email públicos en modelo | User.java | ~5-7 | Encapsulamiento | Medio |
| 7 | No se cierran conexiones JDBC | UserRepository.java | General | Gestión de recursos | Alto |
| 8 | Violación de SRP en AuthService | AuthService.java | General | SOLID – SRP | Medio |
| 9 | Validación de contraseña extremadamente débil | AuthService.java | ~42 | Seguridad – Validación insuficiente | Medio |
| 10 | Uso de System.out.println para logging | AuthService.java | ~24 | Buenas prácticas | Bajo |

---

## Fase 3 - Pruebas Funcionales

### Prueba 1 — Login válido

**Datos sensibles en la respuesta:**

- `"user": "admin"` → expone el nombre del usuario autenticado.  
- `"hash": "827ccb0eea8a706c4c34a16891f84e7b"` → expone el hash de la contraseña.

**¿Debería retornarse eso?**

No. En un endpoint de login nunca se debe devolver el hash de la contraseña ni información interna relacionada con la autenticación.

---

### Prueba 2 — SQL Injection

**¿Qué ocurrió?**

El carácter `'` intenta cerrar la cadena dentro de la consulta SQL y `--` comenta el resto de la instrucción. Esto es una técnica clásica para alterar la consulta y saltarse la validación de contraseña.  
Aunque la autenticación falló, el sistema procesó la entrada sin bloquear caracteres peligrosos y continúa devolviendo información sensible.

**¿Por qué es peligroso en producción?**

1. El sistema sigue devolviendo el hash incluso cuando el login falla, exponiendo información interna que facilita ataques de fuerza bruta o análisis de credenciales.  
2. Aceptar caracteres como `'` y `--` sin usar consultas preparadas indica una vulnerabilidad crítica de SQL Injection, que está dentro del Top 10 de OWASP.

---

### Prueba 3 — Registro con contraseña débil

**Request rechazado:**

http POST "http://localhost:8080/register?u=test&p=123&e=test@test.com"

**¿Es esa una validación suficiente?**

No, no es una validación suficiente. Lo que actualmente estás haciendo parece limitarse únicamente a verificar la longitud de la contraseña, por ejemplo, aceptar si tiene cuatro caracteres y rechazar si tiene tres, lo cual es claramente insuficiente para un sistema de registro real. Tambien existen varios problemas de seguridad importantes: no hay una validación robusta de la contraseña (como exigir mayúsculas, minúsculas, números o caracteres especiales), se están enviando el usuario y la contraseña por query params, lo cual es inseguro, no se valida correctamente el formato del correo electrónico, no hay control para evitar usuarios duplicados y, además, no se está aplicando hashing a la contraseña antes de almacenarla, lo que representa un riesgo crítico de seguridad.









