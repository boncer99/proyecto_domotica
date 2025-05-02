# 🎯 **Objetivo General**

**Diseñar y construir un agente inteligente especializado**, capaz de:

- 🛠️ **Utilizar múltiples herramientas personalizadas** (y opcionalmente herramientas de terceros)  
- 🤖 **Ejecutar tareas de forma autónoma y eficiente**
- 💻 Ser **funcional desde un entorno programático** (por ejemplo, en Google Colab o scripts locales)
- 🌐 **Incluir una interfaz web** (opcional) desplegada en **[Vercel](https://vercel.com/)** como valor agregado


## **1. Diagrama de arquitectura.**
El sistema permite a los usuarios interactuar mediante lenguaje natural para realizar consultas de información en una base de datos. Utiliza herramientas personalizadas (tools) para identificar la tabla adecuada, generar y ejecutar una consulta SQL,  y convertir el resultado en una respuesta. Todo el proceso se realiza mediante un agente basado en LangChain y la API de OpenAI. 


<p align="center">
  <img src="https://github.com/user-attachments/assets/e67128a2-0065-4e84-aea2-357f9fbc8acc" width="600"/>
</p>

## **2	Descripción de herramientas y funciones.**
### **3.1	get_schema(question: str) -> str**
El propósito es recuperar el esquema (estructura) de una tabla específica de la base de datos, a partir de una pregunta. Para hacerlo se llama a la función seleccionar_tabla_v2() para inferir el nombre de la tabla más adecuada según la pregunta. Posteriormente verifica si la tabla existe en la base de datos utilizando db_data.get_table_names(). 
En caso la tabla existe, devuelve su esquema mediante db_data.get_table_info([tabla]), pero si no existe, retorna un mensaje de error indicando las tablas válidas.
### **3.2	seleccionar_tabla_v2(question: str) -> str**
Se utiliza para inferir el nombre de la tabla más relevante en función de una pregunta del usuario. Para ello realiza la consulta sobre todas las tablas disponibles en la base de datos y genera un prompt para el modelo de lenguaje con la pregunta del usuario y las tablas disponibles.
El modelo genera como respuesta el nombre de la tabla sugerida y se valida que dicha tabla exista en la base de datos antes de retornarla.
### **3.3	generar_sql(question: str) -> str**
Se encarga de generar una consulta SQL válida basada en la pregunta del usuario y el esquema de la tabla relevante.  Para ello se vuelve a utilizar la función seleccionar_tabla() para y se obtiene el esquema mediante db_data.get_table_info(). 
Posteriormente se usa una cadena (sqlchain) que incluye una plantilla de prompt que genera la consulta SQL con el modelo de lenguaje, devolviendo la consulta generada como una cadena de texto.
### **3.4	run_query(query: str) -> str**
Se utiliza para ejecutar una consulta SQL directamente en la base de datos y retornar los resultados, mediante el objeto db_data, como una instancia de SQLDatabase. Posteriormente ejecuta query utilizando db_data.run(query) que devuelve los resultados obtenidos, los cuales pueden estar vacíos si no hay registros coincidentes.
### **3.5	generar_respuesta(question: str, sql_query: str, response: str) -> str**
Se encarga de generar una respuesta en lenguaje natural basada en la pregunta original, la consulta SQL generada y los resultados de la base de datos.
Se identifica la tabla relacionada con la pregunta mediante seleccionar_tabla() y se utiliza una plantilla (sqlnatural_chain) para que el modelo de lenguaje genere una explicación natural combinando la pregunta, la SQL generada y la respuesta obtenida, devolviendo una respuesta comprensible para el usuario final.
