# Cómo ver los logs del error

## Paso a paso:

1. En la página actual de GitHub Actions
2. En el panel izquierdo, en la sección "Jobs"
3. Haz clic en: **"Deploy to EC2"** (el que tiene ❌ roja)
4. Se abrirán los logs detallados
5. Busca líneas en rojo o mensajes de error
6. Comparte una captura de esos logs

El error probablemente está relacionado con:
- Conexión SSH al EC2
- Secret LIDRKEY mal configurado
- Permisos en el EC2
- Security Group bloqueando GitHub Actions

Necesito ver el mensaje de error exacto para saber cómo arreglarlo.

