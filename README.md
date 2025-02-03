
# 📌 Documentación: Result Pattern en JavaScript/React 

## Índice

1. **Introducción al Result Pattern**
   - 1.1. ¿Qué es el Result Pattern?
   - 1.2. Definición, orígenes e inspiración (Rust, Go, C#)
   - 1.3. Objetivo principal: manejo explícito de errores sin throw/try-catch
   - 1.4. Comparación con enfoques tradicionales (ventajas y desventajas)
   - 1.5. Relevancia en JavaScript/React:
     - Operaciones asíncronas (`async/await`)
     - Centralización del manejo de estados complejos (ej.: `useReducer`)
     - Mejora en legibilidad y mantenibilidad del código

2. **Estructura del Result Pattern en el Proyecto**
   - 2.1. Objeto de Resultado Estándar
   - 2.2. Ejemplos de Uso (casos de éxito y error)
   - 2.3. Flujo de Operaciones (diagrama Mermaid)
   - 2.4. Campos personalizados en el Resultado

3. **Implementación en Capas Clave**
   - 3.1. Capa de Manejo de Errores Centralizado (`handleApiError`) y el nuevo **Standardized Error Pattern**
   - 3.2. Capa de Contexto (`useReducer`)
   - 3.3. Ejemplos de retorno de datos específicos en funciones asíncronas

4. **Consumo en Componentes React**
   - 4.1. Ejemplo: Redirección sin esperar al estado global
   - 4.2. Gestión de estado local en componentes

5. **Ventajas del Enfoque**
   - 5.1. Claridad y legibilidad en el flujo de control
   - 5.2. Flexibilidad en el consumo de APIs
   - 5.3. Reducción de complejidad en el estado global

6. **Consideraciones y Buenas Prácticas**
   - 6.1. Estructura de objetos y consistencia
   - 6.2. Seguridad y manejo de datos sensibles
   - 6.3. Testing y simulación de resultados
   - 6.4. Diseño de respuestas de la API y contratos claros
   - 6.5. Documentación de campos personalizados
7. Casos de Uso Avanzados
   - 7.1 Manejo de Errores en Flujos Asíncronos Concurrentes
   - 7.2 Uso del Patrón en Microservicios
   - 7.3 Manejo de Errores en Operaciones Transaccionales
8. **Conclusión**
   - 7.1. Resumen de ventajas
   - 7.2. Adaptabilidad y escalabilidad del patrón
   
---

## 1. Introducción al Result Pattern

### 1. 1 ¿Qué es el Result Pattern?
Un patrón de diseño inspirado en Rust, Go y C# que encapsula el resultado de operaciones en un objeto estructurado con:
- **Éxito explícito**: `success: boolean` || `isOk: boolean` 
- **Mensajes descriptivos**: `message: string`
- **Datos relevantes**: Campos personalizados (ej: `idUser`, `statusCode`)

### ¿Por qué usarlo en React?
- **Elimina `try/catch` anidados**: Flujo lineal y legible
- **Centraliza el manejo de errores**: Lógica consistente en toda la app
- **Facilita el estado global**: Integración natural con `useReducer`

### 1.2. Definición, orígenes e inspiración
Inspirado en lenguajes como **Rust**, **Go** y **C#**, el *Result Pattern* se utiliza para evitar el uso excesivo de `throw` y `try-catch`, promoviendo un manejo explícito de los resultados.

> [!NOTE]
>Nota: Aunque el patrón proviene de otros lenguajes, en JavaScript se adapta para trabajar con promesas y async/await.

### 1.3. Objetivo principal
Proveer un mecanismo para retornar el resultado de una operación (**éxito o error**) en un objeto estándar, permitiendo que el consumidor de la función decida cómo manejar cada caso, sin depender de excepciones.

### 1.4. Comparación con enfoques tradicionales

#### 🔴 **Enfoque tradicional (try-catch)**
```javascript
// UserContex.jsx (función reutilizable dentro de un estado global)
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error("Error al obtener el usuario");
    }
    try {
      const proccessData = proccess(data)
      return proccessData
    } catch (e) {
      console.error("Error al procesar la información", e)
    }
    // return await response.json();
  } catch (error) {
    console.error("Error de red", error);
    return null; // ❌ No hay una forma clara de manejar este error
  }
}
```
🔹 **Desventajas:**
- Los errores son capturados dentro de `catch`, lo que puede hacer que los errores pasen desapercibidos.
- La función devuelve `null`, lo cual no indica explícitamente el motivo del error.
- No hay un contrato claro de respuesta.
- Posible necesidad de anidar try-catch debido a manejo de diferentes tipos de procesos

<details> <summary>Ver Ejemplo función proccessData✅</summary>
   
```javascript
// Simulamos una función que llama a una API externa para procesar datos.
async function processDataApi(data) {
  // Simula una llamada a la API y su respuesta
  // Por ejemplo, se podría utilizar axios, pero aquí usamos fetch para simplificar.
  const response = await fetch('https://api.example.com/process', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response; // Se asume que response tiene .status y .json()
}

async function processData(data) {
  try {
    // Llamamos a la API de procesamiento
    const response = await processDataApi(data);
    
    // Procesamos la respuesta de la API
    if (response.status === 200) {
      const processedData = await response.json();
      return {
        success: true,
        message: "Datos procesados correctamente",
        data: processedData
      };
    } else {
      // Si el estado no es 200, consideramos que hubo un error en la API
      return {
        success: false,
        message: `Error en el procesamiento: estado ${response.status}`,
        status: response.status
      };
    }
  } catch (error) {
    // Capturamos errores de red o excepciones internas
    return {
      success: false,
      message: error.message || "Error inesperado en el procesamiento de datos",
      status: error.status || 500
    };
  }
}

```
</details>

#### 🟢 **Enfoque con Result Pattern** (Async Function)
```javascript
// UserContext.jsx (función reutilizable dentro de un estado global)

import { getUserApi } from '@api/users.api' // api externa con axios
export const UsersContext = createContext(null)

export const UsersProvier = ({ children }) => {
  const initialState = { users: [], count: 0 }
  const [stateUser, dispatch] = useReducer(UsersReducer, initialState)
  
  // getUsersContext, createUserContext...

  export const getUserContext = async (userId) => {

   try {
      const res = await getUserApi(userId)

       // o primero podría proccesar la info > Llamamos a la función processData para procesar los datos sin necesidad de anidar aquí un try-catch adicional.
       // const processedResult = processData(res.data);
       // if (!processedResult.success) {
         // Si el procesamiento falla, se retorna el error.
         // Opcionalmente, se puede utilizar handleApiError para transformar el error, si processedResult.error existe.
         // return handleApiError(processedResult.error) || processedResult;
       // }
        // Si todo es exitoso, se retorna el resultado final con los datos procesados.
       // return { 
         // success: true, 
         // message: 'Usuario procesado correctamente', 
         // data: processedResult.data 
        // };
      if (res.status === 200) {
        dispatch({
          type: 'GET_USER',
          payload: res.data
        })
        return { success: true, message: res.data.message }
      }
      
       return { success: false, message: res.data.message ?? 'Error inesperado' }
    }  catch (error) {
      return { success: false, message: res.data.error ?? 'Error inesperado' } 
      // return handleApiError(error) > ya que puede ser .error | .detail | etc.. o información de error extensa
      // en lugar de empezar a introducir múltiples throw new Error aquí
    }
  
  }
  // retorno de la función 
}
// otro componente utilizando la función del contexto

const getUserById = async (id) => {
  
  const { success, message, userId } = await getUserContext(id)
  if (success) {
    // mostrar el mensaje con toast 
  } else {
    // toast.error(message)
  }
}

```

> [!NOTE]
> Las funciones async siempre devuelven una promesa, incluso si no se especifica explícitamente. 
> Retornar un valor dentro de un try o catch equivale a resolver la promesa (Promise.resolve()).
> Lanzar un error (throw) dentro de un catch equivale a rechazar la promesa (Promise.reject()).

✅ **Ventajas:**
- Se evita el uso de `try-catch` innecesario dentro del llamada de la función compartid por el contexto Global.
- El objeto de resultado es explícito (`success: true/false`).
- Se pueden agregar mensajes y códigos de estado personalizados.
- Se puede extender a una utilidad/función encargada de recopilar datos de error útiles (ej: return handleApiError(error) )

### 1.5. Relevancia en JavaScript/React

- 🔹 **Asincronía:** Se integra de forma natural con `async/await`.
- 🔹 **Gestión de estados:** Es ideal para contextos que usan `useReducer` en React, centralizando el manejo de datos y errores.
- 🔹 **Legibilidad:** Mejora la claridad del código al separar la lógica de manejo de errores de la lógica de negocio.
---
<details><summary><strong>Patrón de Diseño: Promise con Resolve/React</strong></summary>

El patrón de diseño basado en Promesas con `resolve`/`reject`es un enfoque tradicional en JavaScript para manejar operaciones asíncronas. Este patrón utiliza el objeto **Promise** para representar un valor que puede estar disponible ahora, en el futuro o nunca. Las funciones que implementan este patrón resuelven (`resolve`) cuando la operación tiene éxito o rechazan (`reject`) cuando ocurre un error.

### Ventajas
- 1 **Familiaridad**: Es ampliamente utilizado en JavaScript, por lo que muchos desarrolladores están familiarizados con él.
- 2 **Separación clara entre éxito y error**: **resolve** indica éxito, mientras que **reject** indica un error. Esto facilita el manejo diferenciado de ambos casos.
- 3 **Compatibilidad con `async`/`await`**: Las promesas son compatibles con async/await, lo que simplifica su uso en funciones asíncronas.
---
### Desventajas
- 1 **Dependencia de Excepciones**: Requiere el uso de `try-catch` o .then / .catch() (cuando se consumen) para manejar errores, lo que puede llevar a código más complejo y menos predecible.
- 2 **Falta de consistencia**: No hay un contrato estándar para los resultados. Cada función puede devolver diferentes estructuras, lo que dificulta la integración.
- 3 **Anidamiento innecesario**: En aplicaciones grandes, especialmente en React, el uso de multiple de try-catch puede llevar a múltiples niveles de anidamiento.

### Ejemplo de código
   
```javascript 
function buscarEnBaseDeDatos(id) {
  return new Promise((resolve, reject) => {
    try {
      const resultado = BaseDatos.buscar(id);
      if (resultado !== null) {
        resolve(resultado); // resolver Éxito
      } else {
        resolve(0); // Indica que no se encontró el registro
      }
    } catch (error) {
      reject("Error al acceder a la base de datos"); // Error
    }
  });
}
// Consumo con Promesas
buscarEnBaseDeDatos(123)
  .then((resultado) => {
    if (resultado === 0) {
      console.log("No se encontró el registro");
    } else {
      console.log("Registro encontrado:", resultado);
    }
  })
  .catch((error) => {
    console.error("Error:", error);
  });
```
</details>

## 2. Estructura del Result Pattern en el Proyecto

### 2.1. Objeto de Resultado Estándar

```javascript
{
  success: boolean,   // Indica si la operación fue exitosa o fallida
  message: string,    // Mensaje descriptivo
  data?: any,         // Datos adicionales (opcional)
  status?: number     // Código de estado HTTP (opcional)
}
```

### 2.2. Ejemplos de Uso

✅ **Caso de éxito:**
```javascript
{ success: true, message: "Usuario creado", data: { id: 123 } }
```

❌ **Caso de error:**
```javascript
{ success: false, message: "Error de red", status: 500 }
```

---

## 3. Implementación en Capas Clave

### 3.1. Capa de Manejo de Errores Centralizado (`handleApiError`)  y el nuevo Standardized Error Pattern

<details> <summary>Ver Diagrama de ejemplo✅</summary>
<image src='https://github.com/user-attachments/assets/8be1b2e5-0369-4e67-b09d-dabf7435c581' />
</details> 

- 🔹 **Propósito:**:   
- Este nuevo patrón se encarga de transformar diversas fuentes de error, ya sea que provengan de la respuesta de una API (por ejemplo, `error.response.data.error`, `error.response.data.message` o `error.response.data.detail`), o de casos en los que el error proviene de `error.request`, o incluso errores generados automáticamente durante la configuración de la solicitud.  
- Es especialmente útil en etapas tempranas de desarrollo, cuando las APIs aún pueden no devolver errores sólidos o uniformes.  
-  Este enfoque complementa al Result Pattern agregando el `message` obtenido mediante extractores/limpiadores de errores y siempre incluye un `status` asociado. El retorno siempre seguirá un formato similar a:  
  `return { success: false, message: extractedMessage, status: statusValue }`.

🔹 **Ventajas del Standardized Error Pattern:**
- **Consistencia:** Garantiza que todos los errores tengan un formato unificado.
- **Claridad:** Proporciona mensajes claros extraídos de diversas propiedades (como `error`, `message`, o `detail`).
- **Robustez durante el desarrollo:** Útil cuando las APIs retornan errores no uniformes, asegurando que siempre se devuelva un `status` junto con el mensaje.
- **Integración:** Se complementa naturalmente con el Result Pattern al retornar un objeto que incluye `success`, `message` y `status`.
🔹 **Manejo de errores HTTP y de red:**
- **Facilitar el manejo de errores en la UI**(ej: modal/toast del `message` de error para el usuario): Los componentes pueden procesar un objeto de error consistente, sin tener que adivinar de qué fuente proviene el problema.


<details> <summary>Ver ejemplo✅</summary>

```javascript
function handleApiError(error) {
  // respues de servidor
  if (error.response) {
  console.error({ ERROR_RESPONSE: error.response})
  const { status, data } = error.response
  // Función helper para formatear mensajes de error
    const formatErrorMessage = (message) => {
      return typeof message === 'string'
        ? message
        : Array.isArray(message)
          ? message[0]
          : JSON.stringify(message)
    }

    // Función para manejar errores anidados
    const extractNestedError = (errorData) => {
      if (typeof errorData === 'string') return errorData
      if (Array.isArray(errorData)) return errorData[0]
      if (typeof errorData === 'object') {
        const firstValue = Object.values(errorData)[0]
        return formatErrorMessage(firstValue)
      }
      return null
    }

    // Prioridad en el manejo de mensajes (puede estar en error.response.data.error, data.message o data.detail)
    const errorPriority = [
      { key: 'message', transform: (msg) => msg },
      { key: 'error', transform: extractNestedError },
      { key: 'detail', transform: (msg) => (msg?.length < 100 ? msg : null) },
      {
        key: 'non_field_errors',
        transform: (errors) => (Array.isArray(errors) ? errors[0] : errors)
      }
    ]
    // otros casos
    // Buscar el primer mensaje de error válido según la prioridad
    // Si es un objeto de errores de validación
    // Mensajes predeterminados según código HTTP
    const httpErrorMessages = {
      400: 'Los datos proporcionados no son válidos. Por favor, verifica la información.',
      401: 'Sesión expirada. Por favor, vuelve a iniciar sesión.',
      //...
    }
    return { success: false, message: httpErrorMessages[status] ||
        `Error inesperado (${status}). Por favor, intenta nuevamente.`, status }
    if (error.request)  {
    // Error de red o solicitud no completada
    }
    if (error.config) // Error en la configuración de la solicitud
  }
  
  return {
    success: false,
    message: error.message ?? "Error desconocido",
    status: error.status ?? 500
  };
}
```

🔹 **Integración en el Contexto:**
```javascript
export const updateUserContext = async (id, usuario) => {
  try {
    const res = await updateUser(id, usuario);
    if (res.status === 200) {
      // Dispatch y lógica de éxito
      return { success: true, message: res.data.message };
    }
  } catch (error) {
    return handleApiError(error); // Uso centralizado del patrón de error
  }
};
```
</details>
---

### 3.2. Capa de Contexto (`useReducer`)

En el contexto de usuarios, cada función asíncrona retorna un objeto siguiendo el contrato del Result Pattern. Por ejemplo:

```javascript
const createUserContext = async (user) => {
  try {
    const res = await createUser(user);
    if (res.status === 200 || res.status === 201) {
      dispatch({ type: 'CREATE_USER', payload: res.data });
      return { success: true, message: res.data.message, userId: res.data.id };
    }
    return { success: false, message: res.data.error || 'Error inesperado' };
  } catch (error) {
    return {...handleApiError(error), idUser: res.data.id } // opcionalmente podemos incluir más información en este respuesta como el idUser
  }
};

```
> [!SUCCESS]
> Esta función combina el patrón de resultados con la integración a React, permitiendo que el componente actúe según el valor retornado sin lanzar excepciones.

---
### 3.3. Ejemplos de retorno de datos específicos
Permite retornar información adicional que puede usarse directamente en la UI:
```javascript
// Ejemplo en createUserContext
return { 
  ...handleApiError(),
  userId: res.data.id // Dato específico para redirección inmediata
};
```
---
## 4 Consumo en Componentes React
### 4.1 Ejemplo: Redirección sin Estado Global
Un componente puede usar el resultado devuelto para redirigir o mostrar notificaciones sin esperar a que el estado global se actualice:
```jsx
const handleSubmit = async () => {
  const { success, userId } = await createUserContext(formData);
  if (success) {
    navigate(`/users/${userId}`); // Redirección directa usando el ID devuelto sin depender del estado global 🚀
  }
};
```
> [!DANGER]
> En ocaciones una función puede ejecutarse antes de que siquiera el estado global pueda actualizarse, lo que puede llevar problemas, el obtener el id a través del retorno inmediato asegura el id para la ruta.

### 4.2 Gestión de Estado Local en Componentes
Permite actualizar estados locales (como cerrar modales o resetear formularios) basándose en el resultado de la operación:
```jsx
const [isModalOpen, setIsModalOpen] = useState(true);

const handleCreateUser = async () => {
  const { success } = await createUserContext(data);
  if (success) {
    setIsModalOpen(false); // Cierra el modal sin depender del estado global
  }
};
```
---
## 5 Ventajas del enfoque
### 5.1 Claridad y legibilidad
- Cada función retorna explícitamente un objeto que indica el éxito o fallo, eliminando la necesidad de múltiples `try-catch`.
- El flujo de control es más claro y predecible. 
### 5.2 Flexibilidad en el consumo de APIs
- Se pueden incluir campos personalizados (por ejemplo, `idUser`) que permiten actuar de inmediato en la UI (como redirecciones) sin esperar a la actualización global.
### 5.3 Reducción de complejidad en el estado global
- No es necesario reflejar cada cambio de la API en el contexto global si solo un componente requiere la información.
- Esto reduce la sobrecarga y mejora el rendimiento y la mantenibilidad del sistema.
---
## 6 Consideraciones y Buenas Prácticas
### 6.1 Estructura de Objetos
- **Consistencia**: Asegúrate de que todas las funciones retornen un objeto con el mismo formato (success, message, data, etc.). Esto garantiza que los consumidores del patrón no tengan que lidiar con respuestas inconsistentes.
- **Extensibilidad**: Permitir la adición de campos personalizados sin romper el contrato. . Por ejemplo, puedes incluir campos como `userId` o `status` según sea necesario.

> [!WARNING]
> Mantener una estructura consistente es crucial para evitar errores en la integración entre componentes y servicios.

### 6.2 Seguridad
- **Tokens y autenticación**: Utilizar interceptores en Axios para la renovación automática de tokens. Esto asegura que las solicitudes siempre estén autenticadas.
- **Protección de datos**: Evita registrar campos sensibles como contraseñas o información personal en los mensajes de error.

<details> <summary>Ver Ejemplo de Interceptores con Axios✅</summary>

```javascript
// axios.interceptor.js
import axios from 'axios';

const API_URL = Object.freeze({
  desarrolo: 'http://api-extern',
  produccion: 'http://api.v1...', // Nota: útilizar una variable de entorno con la url de producción .env para seguridad
  despliege_local: 'http://ngrok..'
});

/**
 * Crea una instancia personalizada de Axios con interceptores.
 * @param {string} path - El endpoint base para esta instancia (ej: 'users/', 'login/').
 * @returns {AxiosInstance} - Una instancia de Axios configurada.
 */
export const createApiInstance = (path = '') => {
  // Crear una instancia de Axios con la URL base y el path específico
  const apiInstance = axios.create({
    baseURL: `${API_URL.desarrollo}/${path}`, // Path dinámico para cada API
  });

  // 🔧 Interceptor de solicitud: Inyecta el token de autenticación
  apiInstance.interceptors.request.use((config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  });

  // 🔧 Interceptor de respuesta: Maneja errores y renovación de tokens
  apiInstance.interceptors.response.use(
    (response) => response,
    async (error) => {
      if (error.response?.status === 401) {
        // Renovar token automáticamente
        const newToken = await refreshToken();
        localStorage.setItem('authToken', newToken);
        error.config.headers.Authorization = `Bearer ${newToken}`;
        return apiInstance(error.config); // Reintentar la solicitud
      }
      return Promise.reject(error);
    }
  );

  return apiInstance;
};

// user.api.js
import { createApiInstance } from './config/defaultAxiosConfig';

/**
 * Instancia de Axios específica para el módulo de usuarios.
 * Define el endpoint base ('api/users') para todas las solicitudes relacionadas con usuarios.
 */
const userApi = createApiInstance('api/users'); // 👈 Endpoint específico para usuarios

/**
 * Obtiene un usuario por su ID.
 * @param {string} userId - ID del usuario a obtener.
 * @returns {Promise<AxiosResponse>} - Respuesta de la API.
 */
export const getUser = async (userId) => {
  return userApi.get(`/${userId}`); // 👈 Endpoint dinámico basado en el ID
};

// demás funciones...

/**
 * NOTA:
 * Este archivo tiene la única responsabilidad de definir los endpoints específicos
 * para consumir APIs relacionadas con usuarios (ej: 'api/users').
 * 
 * - Los interceptores y la lógica de manejo de tokens están centralizados en `createApiInstance`.
 * - Los endpoints se pasan como parámetros a la configuración de Axios, asegurando que este archivo
 *   sea puramente declarativo y se enfoque en definir las rutas de la API.
 * 
 * Para manejar errores y estados globales, se recomienda importar e integrar estas en un archivo de Contexto global Ej: UserContext.jsx con patrones como:
 * - **Result Pattern**: Encapsula resultados (éxito o error) en objetos estándar.
 * - **Standardized Error Pattern**: Centraliza el manejo de errores con mensajes claros y códigos de estado.
 * 
 * Esto permite que este archivo sea modular, reutilizable y fácil de mantener.
 */
```

> [!TIP]
> 💡Un `endpoint` es un punto de acceso específico en una API al que se envían solicitudes para realizar operaciones (como obtener, crear, actualizar o eliminar datos). Por ejemplo: /api/users es un endpoint para gestionar usuarios.***

</details>
<details>
<summary>Ver: Sin este enfoque 🔴</summary>
   
```jsx
// UserContext.jsx
import axios from 'axios';

export const UsersContext = createContext(null);

export const UsersProvider = ({ children }) => {
  const [users, setUsers] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  // 🔴 Problema: Esta función tiene múltiples responsabilidades. 🤯
  // - Maneja el estado global (users, error, loading).
  // - Realiza la llamada a la API.
  // - Procesa los datos.
  // - Maneja errores directamente aquí.
  const fetchUserData = async (userId) => {
    setLoading(true); // Estado de carga mezclado con lógica de negocio.
    try {
      const response = await axios.get(`/api/users/${userId}`);
      if (!response.data) {
        throw new Error("Datos inválidos"); // ❌ Lanzamiento de excepciones innecesario.
      }

      // Procesamiento de datos dentro de la misma función.
      const processedData = processData(response.data);

      // Actualización del estado global directamente aquí.
      setUsers((prevUsers) => [...prevUsers, processedData]);
      setError(null);
    } catch (err) {
      // ❌ Manejo de errores dentro del mismo bloque.
      console.error("Error al obtener usuario:", err.message);
      setError(err.message || "Error desconocido"); Mensajes de error inconsistentes.
    } finally {
      setLoading(false); // Estado de carga mezclado con lógica de negocio.
    }
  };

  // Función adicional para procesar datos, pero está acoplada a la lógica principal.
  const processData = (data) => {
    if (!data.name || !data.email) {
      throw new Error("Datos incompletos"); // ❌ Más lanzamiento de excepciones.
    }
    return { id: data.id, name: data.name.toUpperCase(), email: data.email };
  };

  return (
    <UsersContext.Provider value={{ users, error, loading, fetchUserData }}>
      {children}
    </UsersContext.Provider>
  );
};
```
### Problemas identificados :
- 1 **Múltiples responsabilidades** :
La función fetchUserData maneja el estado global (users, error, loading), realiza la llamada a la API, procesa los datos y maneja errores. Esto viola el principio de separación de responsabilidades . 🤯
- 2 **Manejo de errores inconsistente** :
Los errores se manejan directamente con try-catch, lo que puede llevar a mensajes de error inconsistentes y falta de claridad sobre qué hacer en cada caso. ❌
- 3 **Acoplamiento de lógica** :
La función processData está acoplada a fetchUserData, lo que dificulta reutilizarla en otros contextos. 
- 4 **Estado global sobrecargado** :
El estado global (users, error, loading) se actualiza directamente dentro de la función, lo que puede causar problemas de mantenibilidad y rendimiento si el estado crece. 
- 5 **Falta de contrato claro** :
No hay un formato estándar para los resultados (éxito o error), lo que dificulta su consumo en componentes. ❓
- 6 **Difícil de probar** :
Debido a la mezcla de lógica de negocio, manejo de errores y estado global, es más complicado escribir pruebas unitarias o de integración para esta función. 🛠️
<details/>
   
### 6.3 Testing
- **Mocking**: Simular respuestas con success: true y success: false para probar la lógica de los componentes.
- **Pruebas de errores**: Verificar que los mensajes y la lógica de manejo de errores sean correctos.
#### 6.3.1 Testing con Vitest
`Vitest` es una herramienta de testing moderna y rápida, compatible con `Vite`. Es ideal para proyectos que utilizan frameworks como **React**, **Vue** o **Svelte**.
A continuación, se muestran ejemplos de cómo probar funciones que usan el Result Pattern con Vitest.
<details> <summary>Ver Ejemplo de Testing con Vitest✅</summary>

```javascript
// users.test.ts
import { describe, it, expect, vi } from 'vitest';
import { getUserContext } from './UserContext';
import { getUserApi } from '@api/users.api';

vi.mock('@api/users.api', () => ({
  getUserApi: vi.fn(),
}));

describe('getUserContext', () => {
  it('should return success result for valid user', async () => {
    getUserApi.mockResolvedValue({ status: 200, data: { id: 'validId', name: 'John Doe' } });
    const result = await getUserContext('validId');
    expect(result).toEqual({
      success: true,
      message: 'Usuario obtenido correctamente',
      data: { id: 'validId', name: 'John Doe' },
    });
  });

  it('should return error result for invalid user', async () => {
    getUserApi.mockRejectedValue({ response: { status: 404, data: { message: 'User not found' } } });
    const result = await getUserContext('invalidId');
    expect(result).toEqual({
      success: false,
      message: 'User not found',
      status: 404,
    });
  });
});
```
</details>

### 6.3.2 Nuevas Características de Promesas
El uso de métodos avanzados de promesas como **Promise.allSettled()** y **Promise.any()** puede mejorar significativamente la robustez de tu código al manejar múltiples operaciones asíncronas.

---
### 6.5. Documentación de Campos Personalizados 🛠️
- Mantener un registro de los campos retornados por cada función del contexto para evitar inconsistencias y sobrecarga de datos.
---
## 7. Casos de Uso Avanzados
### 7.1 Manejo de Errores en Flujos Asíncronos Concurrentes
En aplicaciones modernas, es común tener flujos asíncronos concurrentes (por ejemplo, cargar datos desde varias APIs al mismo tiempo). El **Result Pattern** puede extenderse para manejar estos casos de manera eficiente.

</details><summary>Ver Ejemplo: Combinación de Resultados Concurrentes</summary>

```javascript
// utils.js
export const combineResults = (results) => {
  const hasError = results.some((result) => !result.success);
  if (hasError) {
    return {
      success: false,
      message: "Uno o más procesos fallaron",
      errors: results.filter((result) => !result.success),
    };
  }
  return {
    success: true,
    message: "Todos los procesos completados con éxito",
    data: results.map((result) => result.data),
  };
};

// App.jsx
const fetchData = async () => {
  const [userResult, orderResult] = await Promise.all([
    getUserContext("validId"),
    getOrderContext("orderId"),
  ]);

  const combinedResult = combineResults([userResult, orderResult]);
  if (!combinedResult.success) {
    console.error(combinedResult.errors);
  } else {
    console.log(combinedResult.data);
  }
};
```
Explicación:
**Combinación de resultados** : La función combineResults agrupa los resultados de múltiples llamadas y determina si hubo algún error.
**Manejo centralizado** : Si hay errores, se devuelve un objeto con detalles sobre qué procesos fallaron.

</details>

### 7.2 Uso del Patrón en Microservicios
En arquitecturas de microservicios, el **Result Pattern** puede ser útil para estandarizar las respuestas entre servicios y facilitar el manejo de errores distribuidos.

<details/><summary>Ver Ejemplo: Gateway de API</summary>

```javascript
// api-gateway.js
export const callService = async (serviceUrl, payload) => {
  try {
    const response = await fetch(serviceUrl, {
      method: "POST",
      body: JSON.stringify(payload),
    });

    if (!response.ok) {
      throw new Error(`Service error: ${response.status}`);
    }

    const data = await response.json();
    return { success: true, message: "Success", data };
  } catch (error) {
    return handleApiError(error); // Centraliza el manejo de errores
  }
};

// Usage in a gateway
const processRequest = async (request) => {
  const userServiceResult = await callService("http://user-service", request.user);
  const orderServiceResult = await callService("http://order-service", request.order);

  const combinedResult = combineResults([userServiceResult, orderServiceResult]);
  return combinedResult;
};
```
- **Explicación:**
- **Estandarización** : Cada servicio retorna un objeto siguiendo el **Result Pattern** , lo que facilita la integración.
- **Centralización** : El manejo de errores se realiza en un solo lugar (`handleApiError`), reduciendo la duplicación de código.
</details>

### 7.3 Manejo de Errores en Operaciones Transaccionales
En sistemas donde las operaciones deben ser atómicas (todas las operaciones tienen éxito o ninguna), el **Result Pattern** puede combinarse con patrones como **Saga** para revertir cambios en caso de errores.
<details><summary>Ver Ejemplo: Transacciones con Rollback</summary>
   
```javascript
const executeTransaction = async (operations) => {
  const results = [];
  try {
    for (const operation of operations) {
      const result = await operation();
      if (!result.success) {
        throw new Error("Transaction failed");
      }
      results.push(result);
    }
    return { success: true, message: "Transaction completed", data: results };
  } catch (error) {
    // Rollback logic
    for (const result of results) {
      if (result.rollback) {
        await result.rollback();
      }
    }
    return { success: false, message: error.message };
  }
};
```
- **Explicación:**
- **Atomicidad** : Si alguna operación falla, se revierten todas las operaciones previas.
- **Flexibilidad** : Cada operación puede incluir lógica de rollback personalizada.
</details>

## 8. Conclusión

🔹 **Resumen de ventajas:**
- 🔄 **Evita `try-catch` innecesarios**.
- ✅ **Manejo centralizado y explícito de errores**.
- 📈 **Mejor estructura para estados y promesas**.

🔹 **Adaptabilidad y escalabilidad:**

---

**Referencia Adicional:**  

[Goodbye Exceptions: Mastering Error Handling in JavaScript with the Result Pattern](https://dev.to/gautam_kumar_d3daad738680/goodbye-exceptions-mastering-error-handling-in-javascript-with-the-result-pattern-26kb#:~:text=The%20Result%20Pattern%20is%20a,explicitly%20indicates%20success%20or%20failure.)

[Lemon Code🍋 - JavaScript Asíncronico La Guía definitiva](https://lemoncode.net/lemoncode-blog/2018/1/29/javascript-asincrono)

---

*Documentación generada y mejorada a partir de los archivos de un proyecto Desarrollado y las mejores prácticas actuales (o tendencias ) en el desarrollo en React.*
```
