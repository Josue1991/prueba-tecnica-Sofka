# 🚀 Guía de Inicio Rápido - Proyecto Refactorizado

## 📦 Instalación

### 1. Instalar dependencias
```bash
npm install
```

### 2. Iniciar el servidor de desarrollo
```bash
npm start
```

### 3. Compilar para producción
```bash
npm run build
```

---

## 🏗️ Nueva Arquitectura

El proyecto ha sido completamente refactorizado siguiendo **Clean Architecture** y **Principios SOLID**.

### 📁 Estructura de Carpetas

```
src/app/
├── core/                           # Núcleo de la aplicación
│   ├── domain/                     # Capa de Dominio
│   │   ├── entities/               # Entidades de negocio
│   │   │   └── producto-financiero.entity.ts
│   │   ├── repositories/           # Interfaces de repositorios
│   │   │   └── producto-financiero.repository.interface.ts
│   │   └── index.ts
│   │
│   ├── application/                # Capa de Aplicación
│   │   ├── use-cases/              # Casos de uso (lógica de negocio)
│   │   │   ├── get-all-productos.use-case.ts
│   │   │   ├── get-producto-by-id.use-case.ts
│   │   │   ├── create-producto.use-case.ts
│   │   │   ├── update-producto.use-case.ts
│   │   │   └── delete-producto.use-case.ts
│   │   └── index.ts
│   │
│   ├── infrastructure/             # Capa de Infraestructura
│   │   ├── repositories/           # Implementaciones concretas
│   │   │   └── producto-financiero-http.repository.ts
│   │   └── index.ts
│   │
│   └── architecture-diagram.ts     # Diagrama visual de la arquitectura
│
├── shared/                         # Código compartido
│   ├── services/                   # Servicios auxiliares
│   │   ├── pagination.service.ts   # Servicio de paginación
│   │   ├── filter.service.ts       # Servicio de filtrado
│   │   └── sort.service.ts         # Servicio de ordenamiento
│   └── index.ts
│
└── features/                       # Características/Módulos
    └── productos/                  # Módulo de productos
        └── components/             # Componentes refactorizados
            ├── productos-list/
            ├── productos-create/
            └── productos-edit/
```

---

## 🎯 Principios SOLID Implementados

| Principio | Descripción | Implementación |
|-----------|-------------|----------------|
| **S** - Single Responsibility | Cada clase tiene una sola razón de cambio | Casos de uso específicos, servicios especializados |
| **O** - Open/Closed | Abierto a extensión, cerrado a modificación | Interfaces permiten nuevas implementaciones |
| **L** - Liskov Substitution | Las subclases pueden sustituir a sus clases base | Implementaciones intercambiables del repositorio |
| **I** - Interface Segregation | Interfaces específicas y cohesivas | `IProductoFinancieroRepository` con métodos específicos |
| **D** - Dependency Inversion | Depender de abstracciones, no de concreciones | Casos de uso dependen de interfaces, no de implementaciones |

---

## 🔄 Flujo de la Aplicación

```
Usuario
  ↓
Componente Angular (Presentación)
  ↓ usa
Caso de Uso (Aplicación)
  ↓ depende de
IProductoFinancieroRepository (Dominio - Interfaz)
  ↑ implementa
ProductoFinancieroHttpRepository (Infraestructura)
  ↓ llama a
API Backend
```

---

## 📚 Casos de Uso Disponibles

### 1. `GetAllProductosUseCase`
Obtiene todos los productos financieros.

```typescript
constructor(private getAllProductosUseCase: GetAllProductosUseCase) {}

loadData(): void {
  this.getAllProductosUseCase.execute().subscribe(
    productos => console.log(productos)
  );
}
```

### 2. `GetProductoByIdUseCase`
Obtiene un producto por su ID.

```typescript
constructor(private getProductoByIdUseCase: GetProductoByIdUseCase) {}

getProducto(id: string): void {
  this.getProductoByIdUseCase.execute(id).subscribe(
    producto => console.log(producto)
  );
}
```

### 3. `CreateProductoUseCase`
Crea un nuevo producto.

```typescript
constructor(private createProductoUseCase: CreateProductoUseCase) {}

createProducto(): void {
  const producto = new ProductoFinanciero(
    'id-123',
    'Tarjeta de Crédito',
    'Descripción',
    'logo.png',
    new Date(),
    new Date()
  );
  
  this.createProductoUseCase.execute(producto).subscribe(
    () => console.log('Producto creado')
  );
}
```

### 4. `UpdateProductoUseCase`
Actualiza un producto existente.

```typescript
constructor(private updateProductoUseCase: UpdateProductoUseCase) {}

updateProducto(producto: ProductoFinanciero): void {
  this.updateProductoUseCase.execute(producto).subscribe(
    () => console.log('Producto actualizado')
  );
}
```

### 5. `DeleteProductoUseCase`
Elimina un producto.

```typescript
constructor(private deleteProductoUseCase: DeleteProductoUseCase) {}

deleteProducto(id: string): void {
  this.deleteProductoUseCase.execute(id).subscribe(
    () => console.log('Producto eliminado')
  );
}
```

---

## 🛠️ Servicios Auxiliares

### PaginationService
Maneja la lógica de paginación.

```typescript
constructor(private paginationService: PaginationService) {}

paginateData(): void {
  const paginatedData = this.paginationService.paginate(
    this.allData,
    currentPage,
    pageSize
  );
}
```

### FilterService
Maneja la lógica de filtrado.

```typescript
constructor(private filterService: FilterService) {}

filterData(): void {
  const filteredData = this.filterService.filter(
    this.allData,
    'término de búsqueda',
    ['name', 'description']
  );
}
```

### SortService
Maneja la lógica de ordenamiento.

```typescript
constructor(private sortService: SortService) {}

sortData(): void {
  const sortedData = this.sortService.sort(
    this.allData,
    'name',
    'asc'
  );
}
```

---

## 🔧 Configuración de Inyección de Dependencias

En `app.config.ts` se configura la inversión de dependencias:

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    {
      provide: IProductoFinancieroRepository,
      useClass: ProductoFinancieroHttpRepository
    }
  ]
};
```

Esto permite:
- ✅ Cambiar implementaciones sin modificar código
- ✅ Testear con mocks fácilmente
- ✅ Agregar nuevas implementaciones (LocalStorage, IndexedDB, etc.)

---

## 🧪 Testing

### Testear un Caso de Uso

```typescript
describe('CreateProductoUseCase', () => {
  let useCase: CreateProductoUseCase;
  let mockRepository: jasmine.SpyObj<IProductoFinancieroRepository>;

  beforeEach(() => {
    mockRepository = jasmine.createSpyObj('IProductoFinancieroRepository', ['create']);
    useCase = new CreateProductoUseCase(mockRepository);
  });

  it('debe crear un producto', (done) => {
    const producto = new ProductoFinanciero(/* ... */);
    mockRepository.create.and.returnValue(of(undefined));

    useCase.execute(producto).subscribe({
      next: () => {
        expect(mockRepository.create).toHaveBeenCalled();
        done();
      }
    });
  });
});
```

---

## 📖 Documentación Adicional

- **ARCHITECTURE.md**: Documentación completa de la arquitectura
- **REFACTORING_SUMMARY.md**: Resumen de la refactorización
- **src/app/core/architecture-diagram.ts**: Diagrama visual en código

---

## 🎯 Beneficios de esta Arquitectura

### ✅ Mantenibilidad
- Código limpio y organizado
- Responsabilidades claramente definidas
- Fácil de entender y modificar

### ✅ Testabilidad
- Fácil crear mocks y tests unitarios
- Casos de uso aislados
- Servicios independientes

### ✅ Escalabilidad
- Fácil agregar nuevos casos de uso
- Nuevas implementaciones sin romper código
- Módulos desacoplados

### ✅ Flexibilidad
- Cambiar de HTTP a LocalStorage sin tocar lógica
- Reemplazar implementaciones fácilmente
- Agregar nuevas fuentes de datos

---

## 🚀 Comandos Disponibles

```bash
# Desarrollo
npm start                  # Inicia servidor de desarrollo

# Compilación
npm run build             # Compila para producción

# Testing
npm test                  # Ejecuta tests unitarios
npm run test:coverage     # Tests con cobertura

# Linting
npm run lint              # Revisa código con ESLint
```

---

## 📝 Cambios Principales

### ❌ Antes (Sin Clean Architecture)
```typescript
// Todo en el servicio
class ProductoService {
  constructor(private http: HttpClient) {}
  getProductos() { /* HTTP directo */ }
}

// Componente con múltiples responsabilidades
class ProductosListComponent {
  // Paginación, filtrado, ordenamiento, HTTP
}
```

### ✅ Después (Con Clean Architecture)
```typescript
// Separación de capas
class GetAllProductosUseCase {
  constructor(private repository: IProductoFinancieroRepository) {}
  execute() { return this.repository.getAll(); }
}

// Componente solo presenta
class ProductosListComponent {
  constructor(
    private useCase: GetAllProductosUseCase,
    private paginationService: PaginationService
  ) {}
}
```

---

## 🎉 ¡Listo para usar!

El proyecto está completamente refactorizado siguiendo las mejores prácticas de:
- ✅ Clean Architecture
- ✅ Principios SOLID
- ✅ Separation of Concerns
- ✅ Dependency Inversion
- ✅ Domain-Driven Design

---

**¿Necesitas ayuda?** Consulta los archivos de documentación:
- `ARCHITECTURE.md`
- `REFACTORING_SUMMARY.md`
- `src/app/core/architecture-diagram.ts`
