# 🐾 PetShop — ChatBot de Stock con IA

Sistema de gestión de productos para una veterinaria con un chatbot basado en Inteligencia Artificial. Permite consultar en lenguaje natural el catálogo, precios y stock en tiempo real.

Desarrollado como Trabajo Práctico Final utilizando **VirtualBox**, **DietPi**, **Docker Compose**, **n8n**, **Ollama** y **Google Sheets**, integrando un agente de IA capaz de consultar información en tiempo real sobre el stock y los productos de la veterinaria.

## 🎯 Objetivo

Desarrollar un chatbot inteligente que permita consultar el catálogo de una veterinaria utilizando un modelo LLM local (Ollama) y una fuente de datos externa (Google Sheets), cumpliendo con los requisitos del trabajo práctico: incorporación de IA, contenedores Docker y documentación reproducible.

## 🏗️ Arquitectura

```
Usuario (chat)
   → n8n Chat Trigger
      → AI Agent (Ollama qwen2.5:3b)
         → Tool: Google Sheets (catálogo, vía cuenta de servicio)
         → Memory: contexto de la conversación
      → Respuesta en lenguaje natural
```

| Componente | Rol |
|---|---|
| VirtualBox + DietPi | Máquina virtual liviana que aloja todo el stack |
| Docker / Docker Compose | Orquesta el contenedor de n8n |
| Ollama (`qwen2.5:3b`) | Modelo de lenguaje local, corre directo en DietPi |
| n8n | Workflow: recibe el chat y ejecuta el AI Agent |
| Google Sheets API | Catálogo de productos (autenticación por cuenta de servicio) |

## 📁 Contenido del repositorio

```
├── docker-compose.yml     # Definición del servicio n8n
├── workflow/
│   └── PetShop.json       # Export del workflow de n8n (importable)
└── README.md
```

## ✅ Requisitos previos

- VirtualBox con una VM DietPi (con Avahi, Docker y LXDE instalados).
- Una cuenta de servicio de Google Cloud con la Google Sheets API habilitada y con acceso de lectura al catálogo.

## 🚀 Cómo levantar el entorno

### 1. Instalar DietPi en VirtualBox
Crear la VM e instalar DietPi. Durante la instalación, seleccionar los siguientes componentes desde el instalador:
- **Avahi** (para acceso vía `nombre.local`)
- **Docker**
- **LXDE** (entorno de escritorio liviano)

### 2. Instalar Ollama dentro de la VM
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
Descargar el modelo del chatbot:
```bash
ollama run qwen2.5:3b
```

### 3. Levantar n8n con Docker Compose
```bash
mkdir docker && cd docker
# colocar el docker-compose.yml de este repo dentro de la carpeta
docker compose up -d
```
n8n queda disponible en `http://localhost:5678`.

### 4. Importar el workflow
En n8n: **Menú → Import from File** → seleccionar `workflow/PetShop.json`.

### 5. Conectar Ollama desde n8n
⚠️ **Importante**: como n8n corre en un contenedor Docker y Ollama corre directo en el host (DietPi), hay que usar la siguiente URL en las credenciales del nodo Ollama Chat Model — `localhost` no funciona desde dentro del contenedor:
```
http://host.docker.internal:11434
```

### 6. Conectar Google Sheets
1. Crear una cuenta de servicio en Google Cloud Console con la Google Sheets API habilitada.
2. Compartir la planilla del catálogo con el email de la cuenta de servicio (permiso de lector).
3. En el nodo **Get row(s) in sheet in Google Sheets**, cargar las credenciales con el email y la clave de la cuenta de servicio.

### 7. Activar el workflow y probar
Abrir el chat del workflow en n8n ("Open chat") y escribir, por ejemplo:
```
¿Tenés alimento para perros?
```

## 🗂️ Estructura del Google Sheet

| ID | Producto | Especie | Tipo | Precio | Stock |
|---|---|---|---|---|---|
| 1 | Royal Canin 15kg | Perro | Alimento | 98000 | 5 |
| 2 | Pipeta Frontline | Perro | Medicina | 17000 | 20 |
| ... | ... | ... | ... | ... | ... |

- **Especie**: `Perro` o `Gato`
- **Tipo**: `Alimento`, `Medicina` o `Accesorio`

## 🤖 System Prompt del AI Agent

```
Eres el asistente virtual de una veterinaria.

Tienes acceso a una herramienta de Google Sheets que devuelve el catálogo 
completo con las columnas: ID, Producto, Especie, Tipo, Precio y Stock.

Cuando el usuario pregunte por productos, precios, stock, disponibilidad, 
especies o tipos, debes consultar siempre Google Sheets antes de responder.

Después de recibir el catálogo:
1. Analiza todas las condiciones de la pregunta.
2. Muestra únicamente las filas que cumplan todas las condiciones.
3. "Perro" debe coincidir con Especie = "Perro".
4. "Gato" debe coincidir con Especie = "Gato".
5. "Alimento" o "alimentos" debe coincidir con Tipo = "Alimento".
6. "Medicina", "medicamento" o "medicamentos" debe coincidir con Tipo = "Medicina".
7. "Accesorio" o "accesorios" debe coincidir con Tipo = "Accesorio".
8. Para buscar un producto, acepta coincidencias parciales del nombre.
9. Nunca inventes productos, precios ni stock.
10. Si no hay coincidencias, responde que no encontraste productos que cumplan la consulta.
11. Responde siempre en español, de forma breve y directa.
12. No muestres JSON, llamadas de herramientas ni información interna.
13. Los precios deben mostrarse con el formato $98.000.
```

## 💬 Ejemplo de uso

```
Usuario: ¿tenés alimento para perro?
Bot: Tenemos disponible:
     1. Royal Canin 15kg — $98.000 (stock: 5 unidades)
     ¿Necesitás información sobre otro producto?
```

## 👤 Autor

Santiago — Trabajo Práctico Final, integración de IA con automatización de flujos (n8n) y fuentes de datos externas (Google Sheets).
