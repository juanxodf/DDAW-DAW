# **MANUAL DE SONARQUBE: ANÁL ISIS DE CALIDAD DE CODIGO**

> Hecho con amor por Juan Caravantes y Jesús Aparicio 2ºDAW

## **1. Introducción**

SonarQube, herramienta de analisis de codigo muy util para controlar que el proyecto vaya bien en cuanto a calidad, test, etc.

---

## **2. ¿Qué es SonarQube?**

**SonarQube** es una plataforma open-source de inspección continua de la calidad del código, diseñada para detectar automáticamente:

- **Errores y bugs** en el código
- **Vulnerabilidades de seguridad**
- **Code smells** (malas prácticas)
- **Código duplicado**
- **Complejidad ciclomática excesiva** (mucho bucles anidados)
- **Falta de cobertura de tests**
- **Problemas de mantenibilidad**

**Características principales:**
- ✔ Soporte para múltiples lenguajes: Python, Java, JavaScript, C#, PHP, Go, etc.
- ✔ Integración con sistemas CI/CD (GitHub Actions, Jenkins)
- ✔ Análisis automático en pull requests
- ✔ Reglas de calidad personalizables
- ✔ Dashboard centralizado de métricas

---

## **3. Beneficios y Aplicaciones Prácticas**

| Objetivo | Aplicación en el Desarrollo |
|----------|-----------------------------|
| **Calidad del código** | Detecta errores potenciales antes del despliegue |
| **Seguridad** | Identifica vulnerabilidades como inyecciones SQL, XSS, etc. |
| **Estándares de equipo** | Aplica convenciones (PEP8 para Python) consistentemente |
| **Reducción de deuda técnica** | Señala código difícil de mantener |
| **Cobertura de tests** | Mide el porcentaje de código cubierto por pruebas |

---

## **4. Métricas Clave de SonarQube**

### **4.1 Bugs**
Errores de programación que pueden causar comportamientos inesperados.

**Ejemplo detectado:**
```python
def dividir(a, b):
    return a / b  # Critical: Possible division by zero
```

### **4.2 Vulnerabilities**
Problemas de seguridad que podrían ser explotados.

**Ejemplo detectado:**
```python
import os
os.system("delete " + user_input)  # Critical: Command injection vulnerability
```

### **4.3 Code Smells**
Malas prácticas que dificultan el mantenimiento.

**Ejemplo detectado:**
```python
def procesar_datos(d):
    if d.activo:
        if d.valido:
            if d.procesado == False:
                if d.usuario != None:
                    # ... múltiples if anidados
```

### **4.4 Coverage**
Porcentaje de código ejecutado por tests automatizados.

```
Coverage: 45% → SonarQube recomienda aumentar tests unitarios
```

### **4.5 Duplications**
Fragmentos de código idénticos o muy similares.

**Ejemplo detectado:**
```python
def calcular_area_circulo(radio):
    return 3.1416 * radio * radio

def area_circulo(r):
    return 3.1416 * r * r  # Duplicated code
```

---

## **5. Configuración del Análisis**

### **Ejemplo de configuración con SonarScanner:**
```bash
sonar-scanner \
  -Dsonar.projectKey=mi_proyecto_python \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=tu_token_seguro \
  -Dsonar.python.version=3.9
```

### **Archivo de configuración sonar-project.properties:**
```properties
sonar.projectKey=proyecto_calidad
sonar.projectName=Mi Proyecto Python
sonar.projectVersion=1.0
sonar.sources=src
sonar.tests=tests
sonar.python.coverage.reportPaths=coverage.xml
```

---

## **6. Ejemplos Prácticos de Análisis**

### **6.1 Incumplimiento de PEP8**

**Código original:**
```python
def ProcesarUsuario(NombreUsuario,Edad):
    if Edad>18:
        return True
    else:
        return False
```

**Problemas detectados por SonarQube:**
- ❌ Nombre de función en PascalCase (debe ser snake_case)
- ❌ Variables en PascalCase incorrectas
- ❌ Falta espacios alrededor de operadores
- ❌ Falta docstring
- ❌ Return redundante

**Código corregido:**
```python
def procesar_usuario(nombre_usuario: str, edad: int) -> bool:
    """
    Determina si un usuario es mayor de edad.
    
    Args:
        nombre_usuario: Nombre del usuario
        edad: Edad del usuario
        
    Returns:
        bool: True si es mayor de edad, False en caso contrario
    """
    return edad > 18
```

### **6.2 Código Duplicado**

**Código problemático:**
```python
def calcular_precio_total(productos):
    total = 0
    for producto in productos:
        total += producto.precio * producto.cantidad
    return total

def obtener_total_compra(items):
    suma = 0
    for item in items:
        suma += item.precio * item.cantidad
    return suma  # 85% de duplicación detectada
```

**Solución aplicada:**
```python
def calcular_total(elementos):
    return sum(elemento.precio * elemento.cantidad for elemento in elementos)
```

### **6.3 Complejidad Cognitiva Elevada**

**Código complejo:**
```python
def validar_transaccion(transaccion):
    if transaccion.activa:
        if transaccion.monto > 0:
            if transaccion.usuario.habilitado:
                if transaccion.cuenta.saldo >= transaccion.monto:
                    if transaccion.fecha <= datetime.now():
                        return True
    return False
```

**Métrica reportada:** Cognitive Complexity = 8 (HIGH)

**Código refactorizado:**
```python
def validar_transaccion(transaccion) -> bool:
    condiciones = [
        transaccion.activa,
        transaccion.monto > 0,
        transaccion.usuario.habilitado,
        transaccion.cuenta.saldo >= transaccion.monto,
        transaccion.fecha <= datetime.now()
    ]
    return all(condiciones)
```

### **6.4 Vulnerabilidad de Seguridad**

**Código vulnerable:**
```python
import sqlite3

def buscar_usuario(nombre):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    query = f"SELECT * FROM usuarios WHERE nombre = '{nombre}'"
    cursor.execute(query)  # Critical: SQL injection vulnerability
    return cursor.fetchall()
```

**Problema detectado:** SQL Injection Vulnerability

**Solución segura:**
```python
import sqlite3

def buscar_usuario(nombre: str):
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    query = "SELECT * FROM usuarios WHERE nombre = ?"
    cursor.execute(query, (nombre,))  # Uso de parámetros seguros
    return cursor.fetchall()
```

---

## **7. Interpretación de Resultados**

### **7.1 Dashboard Principal**
- **Quality Gate**: Paso/fallo basado en métricas configuradas
- **Reliability**: Evaluación de bugs (A: 0 bugs, E: >20 bugs)
- **Security**: Nivel de vulnerabilidades detectadas
- **Maintainability**: Evaluación de code smells y deuda técnica

### **7.2 Métricas Importantes**
- **Duplication**: Ideal < 3%
- **Coverage**: Recomendado > 80%
- **Complexity**: Mantener baja complejidad ciclomática
- **Issues**: Categorizadas por Blocker, Critical, Major, Minor, Info

---

## **8. Integración en el Flujo de Desarrollo**

### **8.1 En Integración Continua**
```yaml
# Ejemplo GitHub Actions
name: SonarQube Analysis
on: [push, pull_request]
jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@v4
```

### **8.2 En Revisión de Código**
- Comentarios automáticos en Pull Requests
- Bloqueo de merge si no pasa una minima calidad
- Reportes de nueva deuda técnica

---

## **9. Conclusión**

**SonarQube** se consolida como una herramienta indispensable para garantizar la calidad del código en proyectos de desarrollo software. Su implementación proporciona:

- **Detección temprana** de problemas de calidad y seguridad
- **Estandarización** de prácticas de coding across del equipo
- **Metricas objetivas** para la mejora continua
- **Prevención** de deuda técnica acumulada

La adopción de SonarQube en el ciclo de desarrollo no solo mejora la calidad del producto final, sino que también fomenta mejores prácticas de programación entre los desarrolladores, preparándolos para entornos profesionales donde la calidad del código es un requisito fundamental.