# Manual de Configuración Web y Operación: CI/CD Open Source

Este documento consolida la guía paso a paso para la configuración visual en el navegador, la estructura del proyecto demo (**Basta App** en PHP 8.5) y la matriz completa de resolución de problemas para la infraestructura con **Forgejo** y **Woodpecker CI v3** sobre **Podman (Rootless)**.

---

## 1. Configuración Inicial de Forgejo (Servidor Git)
*Se realiza tras ejecutar el playbook `setup-forgejo.yml`.*

1. **Acceder al portal:** Abre el navegador e ingresa a `http://<IP_SERVIDOR>:3000`.
2. **Instalación Inicial:** Aparecerá la pantalla de configuración general. Mantén los valores predeterminados, baja al final de la página y haz clic en **"Instalar Forgejo"**.
3. **Crear Administrador:** Haz clic en **"Registrarse"** para crear la primera cuenta. Por diseño, el primer usuario registrado obtiene privilegios globales de Administrador (`esanchez`).
4. **Habilitar Webhooks Locales (Paso Crítico de Seguridad):**
   * Forgejo bloquea por defecto las peticiones salientes hacia direcciones IP privadas/locales (SSRF Protection).
   * Para permitir que Forgejo notifique a Woodpecker, ejecuta este comando en el servidor:
     ```bash
     podman exec -it forgejo bash -c "echo -e '\n[webhook]\nALLOWED_HOST_LIST = *' >> /data/gitea/conf/app.ini"
     podman restart forgejo
     ```
5. **Crear Aplicación OAuth2:**
   * Ve a **Configuración** (ícono de engranaje ⚙️ junto a tu usuario).
   * En el menú lateral izquierdo, selecciona **Integraciones > Aplicaciones**.
   * Registra una nueva aplicación con los siguientes datos:
     * **Nombre de la aplicación:** `Woodpecker CI`
     * **URI de redirección:** `http://<IP_SERVIDOR>:8000/authorize`
   * Al guardar, el sistema generará un **ID de Cliente** y un **Secreto de Cliente**. Copia ambos valores para el playbook de Woodpecker.

---

## 2. Enlace e Infraestructura de Woodpecker CI v3
*Se realiza tras ejecutar el playbook `setup-woodpecker.yml`.*

1. **Acceder al portal:** Abre el navegador e ingresa a `http://<IP_SERVIDOR>:8000`.
2. **Inicio de sesión OAuth2:** Haz clic en el botón **"Login"**.
3. **Autorización:** Serás redirigido automáticamente a Forgejo. Haz clic en **"Autorizar aplicación"**.
4. **Sincronización:** De regreso en Woodpecker, haz clic en el botón de recargar **`↻`** para listar los repositorios disponibles en Forgejo.
5. **Habilitar Repositorio:** Busca la aplicación e ingresa para hacer clic en **"Enable"**. Esto registrará el Webhook de forma automática en Forgejo.

---

## 3. Proyecto Demo: "Basta App" en PHP 8.5 (Etapa 4)

Estructura de archivos dentro del servidor en `~/clase/basta-app/`:

```text
basta-app/
├── .woodpecker.yml   # Archivo de pipeline (Sintaxis v3)
├── Game.php          # Lógica del juego en PHP 8.5
├── index.php         # Interfaz web responsiva (Tailwind CSS)
└── test.php          # Suite de pruebas unitarias