# Arquitectura del Proyecto - Clean Architecture & SOLID

## 📐 Arquitectura Implementada

Este proyecto ha sido refactorizado siguiendo los principios de **Clean Architecture** (Arquitectura Limpia) y **SOLID**.

### Estructura de Capas

```
src/app/
├── core/
│   ├── domain/              # Capa de Dominio (Entidades y Contratos)
│   │   ├── entities/        # Entidades de negocio
│   │   └── repositories/    # Interfaces de repositorios
│   │
│   ├── application/         # Capa de Aplicación (Casos de Uso)
│   │   └── use-cases/       # Lógica de negocio
│   │
│   └── infrastructure/      # Capa de Infraestructura (Implementaciones)
│       └── repositories/    # Implementaciones concretas de repositorios
│
├── shared/                  # Servicios compartidos
│   └── services/            # Servicios auxiliares reutilizables
│
└── features/                # Características/Módulos
    └── productos/           # Módulo de productos
        └── components/      # Componentes de presentación
```

---

## 🎯 Principios SOLID Aplicados

### ✅ **S - Single Responsibility Principle (Principio de Responsabilidad Única)**

Cada clase tiene una única razón de cambio:

- **`GetAllProductosUseCase`**: Solo obtiene productos
- **`CreateProductoUseCase`**: Solo crea productos
- **`PaginationService`**: Solo maneja paginación
- **`FilterService`**: Solo maneja filtrado
- **`SortService`**: Solo maneja ordenamiento

**Antes:**
```typescript
// ProductosListComponent tenía TODO: paginación, filtrado, ordenamiento, llamadas HTTP
class ProductosListComponent {
  // 200+ líneas mezclando responsabilidades
}
```

**Después:**
```typescript
// Cada servicio tiene una responsabilidad específica
class PaginationService { /* solo paginación */ }
class FilterService { /* solo filtrado */ }
class SortService { /* solo ordenamiento */ }
```

---

### ✅ **O - Open/Closed Principle (Principio Abierto/Cerrado)**

Las clases están abiertas para extensión pero cerradas para modificación:

```typescript
// Puedes crear nuevas implementaciones del repositorio sin modificar código existente
export abstract class IProductoFinancieroRepository {
    abstract getAll(): Observable<ProductoFinanciero[]>;
    // ...
}

// Implementación actual: HTTP
export class ProductoFinancieroHttpRepository implements IProductoFinancieroRepository {}

// Futuras implementaciones: LocalStorage, IndexedDB, etc.
// sin modificar los casos de uso
```

---

### ✅ **L - Liskov Substitution Principle (Principio de Sustitución de Liskov)**

Las implementaciones pueden ser sustituidas sin romper el código:

```typescript
// Cualquier implementación de IProductoFinancieroRepository
// puede ser usada por los casos de uso sin modificaciones
constructor(private readonly repository: IProductoFinancieroRepository) {}
```

---

### ✅ **I - Interface Segregation Principle (Principio de Segregación de Interfaces)**

Interfaces específicas y cohesivas:

```typescript
// Interfaz específica para operaciones de repositorio
export abstract class IProductoFinancieroRepository {
    abstract getAll(): Observable<ProductoFinanciero[]>;
    abstract getById(id: string): Observable<ProductoFinanciero>;
    abstract create(producto: ProductoFinanciero): Observable<void>;
    abstract update(producto: ProductoFinanciero): Observable<void>;
    abstract delete(id: string): Observable<void>;
}
```

---

### ✅ **D - Dependency Inversion Principle (Principio de Inversión de Dependencias)**

Los módulos de alto nivel no dependen de módulos de bajo nivel, ambos dependen de abstracciones:

**Configuración en `app.config.ts`:**
```typescript
{
  provide: IProductoFinancieroRepository,
  useClass: ProductoFinancieroHttpRepository
}
```

**Uso en casos de uso:**
```typescript
// Los casos de uso dependen de la abstracción, no de la implementación
export class GetAllProductosUseCase {
    constructor(private readonly repository: IProductoFinancieroRepository) {}
}
```

**Flujo de dependencias:**
```
Componente → Caso de Uso → IRepository (Abstracción) ← HttpRepository (Implementación)
```

---

## 🏗️ Clean Architecture - Capas

### 1️⃣ **Capa de Dominio** (`core/domain/`)

Contiene la lógica de negocio central y las reglas empresariales:

- **Entidades**: Objetos de negocio con validación
  ```typescript
  export class ProductoFinanciero {
      constructor(
          public readonly id: string,
          public readonly name: string,
          // ...
      ) {
          this.validate(); // Validación en el dominio
      }
  }
  ```

- **Interfaces de Repositorio**: Contratos para acceso a datos
  ```typescript
  export abstract class IProductoFinancieroRepository {
      abstract getAll(): Observable<ProductoFinanciero[]>;
  }
  ```

**Características:**
- ❌ No depende de ninguna otra capa
- ✅ Define las reglas de negocio
- ✅ Independiente de frameworks y tecnologías

---

### 2️⃣ **Capa de Aplicación** (`core/application/`)

Contiene los casos de uso (Use Cases):

```typescript
@Injectable()
export class CreateProductoUseCase {
    constructor(private readonly repository: IProductoFinancieroRepository) {}
    
    execute(producto: ProductoFinanciero): Observable<void> {
        // Validación y lógica de negocio
        return this.repository.create(producto);
    }
}
```

**Características:**
- ✅ Orquesta el flujo de datos
- ✅ Depende solo de la capa de dominio
- ✅ Define qué hace la aplicación

---

### 3️⃣ **Capa de Infraestructura** (`core/infrastructure/`)

Implementaciones concretas de las interfaces del dominio:

```typescript
@Injectable()
export class ProductoFinancieroHttpRepository implements IProductoFinancieroRepository {
    constructor(private readonly http: HttpClient) {}
    
    getAll(): Observable<ProductoFinanciero[]> {
        return this.http.get<{data: any[]}>(`${this.apiUrl}/products`)
            .pipe(map(response => response.data.map(item => this.mapToDomain(item))));
    }
}
```

**Características:**
- ✅ Implementa las interfaces del dominio
- ✅ Maneja detalles técnicos (HTTP, base de datos, etc.)
- ✅ Puede ser reemplazada sin afectar otras capas

---

### 4️⃣ **Capa de Presentación** (`features/`)

Componentes Angular que interactúan con el usuario:

```typescript
export class ProductosListComponent {
    constructor(
        private readonly getAllProductosUseCase: GetAllProductosUseCase,
        private readonly deleteProductoUseCase: DeleteProductoUseCase,
        private readonly paginationService: PaginationService
    ) {}
    
    loadData(): void {
        this.getAllProductosUseCase.execute().subscribe(/* ... */);
    }
}
```

**Características:**
- ✅ Delega la lógica a los casos de uso
- ✅ Solo maneja presentación e interacción
- ✅ Depende de abstracciones

---

## 🔄 Flujo de Datos

```
Usuario
  ↓
Componente (Presentación)
  ↓
Caso de Uso (Aplicación)
  ↓
IRepository (Dominio - Interfaz)
  ↓
HttpRepository (Infraestructura - Implementación)
  ↓
API Externa
```

---

## 🎁 Beneficios de esta Arquitectura

### ✅ **Mantenibilidad**
- Código más limpio y organizado
- Fácil de entender y modificar
- Responsabilidades claramente definidas

### ✅ **Testabilidad**
- Fácil crear mocks de repositorios
- Casos de uso aislados y testeables
- Servicios independientes

### ✅ **Escalabilidad**
- Fácil agregar nuevos casos de uso
- Nuevas implementaciones sin romper código existente
- Módulos desacoplados

### ✅ **Flexibilidad**
- Cambiar de HTTP a LocalStorage sin tocar casos de uso
- Reemplazar implementaciones fácilmente
- Agregar nuevas fuentes de datos

### ✅ **Reusabilidad**
- Servicios compartidos (paginación, filtrado, ordenamiento)
- Casos de uso reutilizables
- Entidades de dominio reutilizables

---

## 📝 Comparación Antes vs Después

### ❌ **Antes (Sin Clean Architecture ni SOLID)**

```typescript
// ProductosListComponent
class ProductosListComponent {
  constructor(private productoService: ProductoService) {}
  
  // Mezcla de responsabilidades:
  // - Llamadas HTTP
  // - Paginación
  // - Filtrado
  // - Ordenamiento
  // - Manejo de estado
  // 200+ líneas
}

// ProductoService con HttpClient acoplado directamente
class ProductoService {
  constructor(private http: HttpClient) {}
  getProductos() { /* HTTP directo */ }
}
```

**Problemas:**
- ❌ Violación de Single Responsibility
- ❌ Acoplamiento fuerte a HttpClient
- ❌ Difícil de testear
- ❌ No hay separación de capas
- ❌ Lógica de negocio en componentes

---

### ✅ **Después (Con Clean Architecture y SOLID)**

```typescript
// Componente - Solo presentación
class ProductosListComponent {
  constructor(
    private getAllProductosUseCase: GetAllProductosUseCase,
    private paginationService: PaginationService,
    private filterService: FilterService
  ) {}
  
  loadData() {
    this.getAllProductosUseCase.execute().subscribe(/* ... */);
  }
}

// Caso de Uso - Lógica de negocio
class GetAllProductosUseCase {
  constructor(private repository: IProductoFinancieroRepository) {}
  execute() { return this.repository.getAll(); }
}

// Repositorio - Implementación
class ProductoFinancieroHttpRepository implements IProductoFinancieroRepository {
  constructor(private http: HttpClient) {}
  getAll() { /* Implementación HTTP */ }
}
```

**Beneficios:**
- ✅ Responsabilidades separadas
- ✅ Dependencias invertidas
- ✅ Fácil de testear
- ✅ Capas bien definidas
- ✅ Lógica de negocio aislada

---

## 🧪 Cómo Testear

### Test de Caso de Uso (con mock del repositorio)
```typescript
describe('GetAllProductosUseCase', () => {
  it('debe retornar todos los productos', () => {
    const mockRepository = jasmine.createSpyObj('IProductoFinancieroRepository', ['getAll']);
    mockRepository.getAll.and.returnValue(of([/* productos */]));
    
    const useCase = new GetAllProductosUseCase(mockRepository);
    useCase.execute().subscribe(/* aserciones */);
  });
});
```

### Test de Servicio
```typescript
describe('PaginationService', () => {
  it('debe paginar correctamente', () => {
    const service = new PaginationService();
    const data = [1, 2, 3, 4, 5];
    const result = service.paginate(data, 1, 2);
    expect(result).toEqual([1, 2]);
  });
});
```

---

## 🚀 Próximos Pasos Recomendados

1. **Testing**: Agregar tests unitarios para cada capa
2. **Validaciones**: Mejorar validaciones en entidades de dominio
3. **Error Handling**: Implementar manejo de errores centralizado
4. **DTOs**: Crear DTOs específicos para transferencia de datos
5. **Mappers**: Extraer lógica de mapeo a clases dedicadas
6. **State Management**: Considerar NgRx o Signals para estado global
7. **Interceptors**: Agregar interceptors HTTP para manejo de errores y tokens

---

## 📚 Referencias

- [Clean Architecture - Robert C. Martin](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [SOLID Principles](https://en.wikipedia.org/wiki/SOLID)
- [Angular Architecture Best Practices](https://angular.io/guide/architecture)

---

## 👨‍💻 Autor

Refactorización implementada siguiendo los principios de Clean Architecture y SOLID.
