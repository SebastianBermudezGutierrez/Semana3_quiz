# ADR-001: Refactorización del módulo de autenticación por vulnerabilidades críticas de seguridad y violaciones de Clean Code

## Contexto

El sistema actual implementa un módulo básico de autenticación con endpoints de login y registro utilizando JDBC manual y hashing MD5. Durante la auditoría técnica se identificaron múltiples vulnerabilidades críticas, incluyendo SQL Injection debido al uso de Statement con concatenación de strings, uso de MD5 como algoritmo de hashing inseguro, exposición del hash de contraseña en respuestas HTTP y credenciales de base de datos hardcodeadas en el código fuente. Además, detectamos violaciones a principios de Clean Code y SOLID, como falta de encapsulamiento en el modelo User, validación débil de contraseñas, uso de System.out.println en lugar de logging estructurado y sobrecarga de responsabilidades en AuthService (violación de SRP).

Estas vulnerabilidades representan un riesgo alto para el negocio y los usuarios, ya que permiten bypass de autenticación, robo de información y posible compromiso total de la base de datos. Es urgente realizar una refactorización antes de considerar el sistema apto para producción.

## Decisión

Se tomarán las siguientes decisiones técnicas:

1. Reemplazar Statement por PreparedStatement para eliminar SQL Injection mediante consultas parametrizadas.

2. Reemplazar el algoritmo MD5 por BCrypt, utilizando un PasswordEncoder robusto que incluya salting automático y resistencia a ataques de fuerza bruta.

3. Eliminar la exposición del hash de contraseña en la respuesta del endpoint /login y retornar únicamente información necesaria (por ejemplo, mensaje de éxito o token).

4. Mover credenciales de base de datos a variables de entorno o archivo de configuración externo.

5. Aplicar encapsulamiento al modelo User utilizando atributos privados y métodos getter/setter.

6. Separar responsabilidades aplicando SRP:
   - Controller: manejo HTTP
   - Service: lógica de negocio
   - Repository: acceso a datos
   - PasswordEncoder: hashing

La arquitectura resultante será más modular, segura y mantenible.

## Consecuencias

### Consecuencias positivas

- Eliminación de vulnerabilidades críticas (SQL Injection)
- Mejora significativa en seguridad de contraseñas
- Reducción de exposición de datos sensibles
- Mejor mantenibilidad y legibilidad del código
- Mayor alineación con principios SOLID
- Código más testeable

### Consecuencias negativas o riesgos

- Tiempo adicional de refactorización
- Posibles regresiones si no se implementan pruebas adecuadas
- Necesidad de migración de contraseñas existentes
- Incremento leve en complejidad técnica

## Alternativas consideradas

1. Reescribir el módulo completamente desde cero  
   → Descartado por mayor costo y riesgo innecesario.

2. Corregir únicamente SQL Injection sin mejorar hashing ni arquitectura  
   → Descartado porque mantendría vulnerabilidades críticas como MD5 y exposición de hash.

3. Mantener MD5 agregando sal manual  
   → Descartado porque MD5 sigue siendo criptográficamente débil frente a estándares modernos.

La decisión final prioriza seguridad, mantenibilidad y reducción de deuda técnica.