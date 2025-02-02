
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

7. **Conclusión**
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
> [!INFO]
> Recordar que el retorno de un async-await devuelve una promesa aúnque no se este retornando explicitamente ej: return > Promise.resolve() ; entonces lo que se retorne dentro del catch es como decir Promise.reject()

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
> [!INFO]
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
> [!INFO]
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
- **Consistencia**: Asegurarse de que todas las funciones retornen un objeto con el mismo formato.
- **Extensibilidad**: Permitir la adición de campos personalizados sin romper el contrato.
### 6.2 Seguridad
- **Tokens y autenticación**: Utilizar interceptores en Axios para la renovación automática de tokens.
- **Protección de datos**: No registrar campos sensibles como contraseñas.
### 6.3 Testing
- **Mocking**: Simular respuestas con success: true y success: false para probar la lógica de los componentes.
- **Pruebas de errores**: Verificar que los mensajes y la lógica de manejo de errores sean correctos.
---
### 6.5. Documentación de Campos Personalizados
- Mantener un registro de los campos retornados por cada función del contexto para evitar inconsistencias y sobrecarga de datos.
---

## 7. Conclusión

🔹 **Resumen de ventajas:**
- 🔄 **Evita `try-catch` innecesarios**.
- ✅ **Manejo centralizado y explícito de errores**.
- 📈 **Mejor estructura para estados y promesas**.

🔹 **Adaptabilidad y escalabilidad:**

---

**Referencia Adicional:**  

[Goodbye Exceptions: Mastering Error Handling in JavaScript with the Result Pattern](https://dev.to/gautam_kumar_d3daad738680/goodbye-exceptions-mastering-error-handling-in-javascript-with-the-result-pattern-26kb#:~:text=The%20Result%20Pattern%20is%20a,explicitly%20indicates%20success%20or%20failure.)

---

*Documentación generada y mejorada a partir de los archivos de un proyecto Desarrollado y las mejores prácticas actuales (o tendencias ) en el desarrollo en React.*
```
