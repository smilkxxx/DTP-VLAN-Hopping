DTP VLAN Hopping Attack — Documentación Técnica

Autor: Alvaro Baez

Matrícula: 2021-1150

Fecha: Junio 2026


Tabla de Contenidos


Objetivo del Laboratorio
Objetivo del Script / Herramienta
Requisitos
Documentación de la Red
Funcionamiento del Ataque
Demostración Paso a Paso
Contramedidas



Objetivo del Laboratorio

Demostrar cómo un atacante puede explotar el protocolo DTP (Dynamic Trunking Protocol) para convertir un puerto de acceso en un puerto troncal, obteniendo visibilidad sobre el tráfico de todas las VLANs de la red — ataque conocido como VLAN Hopping.


Objetivo del Script / Herramienta

Herramienta utilizada: yersinia

Yersinia es un framework de ataques a protocolos de capa 2. En este laboratorio se utiliza específicamente el módulo DTP para enviar tramas de negociación que fuerzan al switch víctima a establecer un enlace troncal con el atacante.

Parámetros usados

bashsudo yersinia dtp -attack 1 -interface eth0

ParámetroDescripcióndtpMódulo de ataque al protocolo DTP-attack 1Tipo 1: enviar paquete DTP para negociar trunk-interface eth0Interfaz de red del atacante hacia el switch

Requisitos para utilizar la herramienta


Sistema operativo: Kali Linux
Herramienta: yersinia (disponible en repos de Kali)
Acceso a la red: el atacante debe estar conectado al mismo switch que la víctima
El puerto del switch debe estar en modo dynamic desirable o dynamic auto


bash# Instalación
sudo apt install yersinia -y


Documentación de la Red

Topología

                    Router-Alvaro
                    (20.21.11.1)
                         |
                      Gi0/1 | Et0/0
                         |
                        IOU2
                       (Switch)
                    /          \
                Et0/1          Et0/2
                  |              |
              Attack           Victima
           (20.21.11.50)    (20.21.11.52)
               Eth0              Eth0

Direccionamiento IP

DispositivoInterfazIPRolRouter-AlvaroGi0/120.21.11.1GatewayIOU2Et0/0—Uplink al routerAttack (Kali)Eth020.21.11.50AtacanteVictima (Kali)Eth020.21.11.52Víctima


Funcionamiento del Ataque

1. Atacante (eth0) conectado a IOU2 puerto Et0/1
          ↓
2. yersinia envía tramas DTP al switch
          ↓
3. IOU2 acepta la negociación (puerto en dynamic desirable)
          ↓
4. Et0/1 pasa de modo ACCESS a modo TRUNK
          ↓
5. Atacante recibe tráfico de TODAS las VLANs ✅

Cuando un puerto está en modo dynamic desirable, el switch responde afirmativamente a cualquier solicitud de negociación DTP. Yersinia se aprovecha de esto enviando tramas DTP falsas que simulan ser un switch legítimo intentando establecer un trunk.


Demostración Paso a Paso

Antes del ataque — Puerto en modo vulnerable

El puerto Et0/1 está configurado en dynamic desirable, lo que permite negociación DTP.

bash! En IOU2
interface Et0/1
 switchport mode dynamic desirable
 no switchport nonegotiate

Mostrar imagen


El puerto muestra Administrative Mode: static access y Negotiation of Trunking: Off — este es el estado de referencia antes de hacer vulnerable el puerto.




Durante el ataque — Yersinia negociando trunk

bash# En Kali (Atacante)
sudo yersinia dtp -attack 1 -interface eth0

Mostrar imagen


Yersinia detecta vecinos DTP. Se observa el cambio de estado ACCESS/DESIRABLE a TRUNK/DESIRABLE, confirmando que el puerto fue convertido a trunk exitosamente.




Después — Contramedida aplicada

bash! En IOU2
interface Et0/1
 switchport mode access
 switchport nonegotiate

Mostrar imagen


Con la contramedida aplicada, el puerto vuelve a static access y Negotiation of Trunking: Off. Yersinia ya no puede negociar trunk.




Contramedidas

Deshabilitar DTP en puertos de acceso

bash! En IOU2 — aplicar en TODOS los puertos de hosts
interface Et0/1
 switchport mode access
 switchport nonegotiate

interface Et0/2
 switchport mode access
 switchport nonegotiate


Nota: El comando switchport nonegotiate solo funciona después de switchport mode access. Aplicarlos en orden incorrecto genera error de conflicto.



Verificación

bashshow interfaces Et0/1 switchport

Salida esperada:

Administrative Mode: static access
Operational Mode: static access
Negotiation of Trunking: Off    ← DTP deshabilitado ✅

¿Por qué funciona?

Al forzar el puerto a static access con nonegotiate, el switch deja de responder a tramas DTP. Aunque yersinia siga enviando paquetes de negociación, el puerto los ignora completamente.


Resumen de Comandos

AcciónComandoHacer vulnerableswitchport mode dynamic desirableLanzar ataquesudo yersinia dtp -attack 1 -interface eth0Verificar trunkshow interfaces Et0/1 trunkAplicar contramedidaswitchport mode access + switchport nonegotiateVerificar protecciónshow interfaces Et0/1 switchport
