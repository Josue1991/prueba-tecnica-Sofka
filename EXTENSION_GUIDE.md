# 🔮 Guía de Extensión - Cómo Agregar Nuevas Funcionalidades

Esta guía muestra cómo extender la aplicación siguiendo Clean Architecture y SOLID.

---

## 📦 Ejemplo 1: Agregar una nueva implementación de repositorio (LocalStorage)

### 1️⃣ Crear la nueva implementación

```typescript
// src/app/core/infrastructure/repositories/producto-financiero-localstorage.repository.ts

import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';
import { IProductoFinancieroRepository } from '../../domain/repositories/producto-financiero.repository.interface';
import { ProductoFinanciero } from '../../domain/entities/producto-financiero.entity';

@Injectable({
    providedIn: 'root'
})
export class ProductoFinancieroLocalStorageRepository implements IProductoFinancieroRepository {
    private readonly STORAGE_KEY = 'productos_financieros';

    getAll(): Observable<ProductoFinanciero[]> {
        const data = localStorage.getItem(this.STORAGE_KEY);
        const productos = data ? JSON.parse(data) : [];
        return of(productos.map((item: any) => ProductoFinanciero.fromPlainObject(item)));
    }

    getById(id: string): Observable<ProductoFinanciero> {
        const data = localStorage.getItem(this.STORAGE_KEY);
        const productos = data ? JSON.parse(data) : [];
        const producto = productos.find((p: any) => p.id === id);
        return of(ProductoFinanciero.fromPlainObject(producto));
    }

    create(producto: ProductoFinanciero): Observable<void> {
        const data = localStorage.getItem(this.STORAGE_KEY);
        const productos = data ? JSON.parse(data) : [];
        productos.push(producto.toPlainObject());
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(productos));
        return of(undefined);
    }

    update(producto: ProductoFinanciero): Observable<void> {
        const data = localStorage.getItem(this.STORAGE_KEY);
        const productos = data ? JSON.parse(data) : [];
        const index = productos.findIndex((p: any) => p.id === producto.id);
        if (index !== -1) {
            productos[index] = producto.toPlainObject();
            localStorage.setItem(this.STORAGE_KEY, JSON.stringify(productos));
        }
        return of(undefined);
    }

    delete(id: string): Observable<void> {
        const data = localStorage.getItem(this.STORAGE_KEY);
        const productos = data ? JSON.parse(data) : [];
        const filtered = productos.filter((p: any) => p.id !== id);
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(filtered));
        return of(undefined);
    }
}
```

### 2️⃣ Cambiar la configuración en app.config.ts

```typescript
// app.config.ts
import { ProductoFinancieroLocalStorageRepository } from './core/infrastructure/repositories/producto-financiero-localstorage.repository';

export const appConfig: ApplicationConfig = {
  providers: [
    // ... otros providers
    {
      provide: IProductoFinancieroRepository,
      useClass: ProductoFinancieroLocalStorageRepository  // ← Cambio aquí
    }
  ]
};
```

**¡Listo!** Ahora toda la aplicación usa LocalStorage sin cambiar ningún caso de uso ni componente.

---

## 🆕 Ejemplo 2: Agregar un nuevo caso de uso

### Escenario: Búsqueda avanzada de productos

### 1️⃣ Crear la interfaz en el repositorio

```typescript
// src/app/core/domain/repositories/producto-financiero.repository.interface.ts

export abstract class IProductoFinancieroRepository {
    abstract getAll(): Observable<ProductoFinanciero[]>;
    abstract getById(id: string): Observable<ProductoFinanciero>;
    abstract create(producto: ProductoFinanciero): Observable<void>;
    abstract update(producto: ProductoFinanciero): Observable<void>;
    abstract delete(id: string): Observable<void>;
    
    // ✨ Nuevo método
    abstract searchByName(name: string): Observable<ProductoFinanciero[]>;
}
```

### 2️⃣ Implementar en el repositorio HTTP

```typescript
// src/app/core/infrastructure/repositories/producto-financiero-http.repository.ts

export class ProductoFinancieroHttpRepository implements IProductoFinancieroRepository {
    // ... métodos existentes

    searchByName(name: string): Observable<ProductoFinanciero[]> {
        return this.http.get<{ data: any[] }>(`${this.apiUrl}?name=${name}`).pipe(
            map(response => response.data.map(item => this.mapToDomain(item))),
            catchError(this.handleError)
        );
    }
}
```

### 3️⃣ Crear el caso de uso

```typescript
// src/app/core/application/use-cases/search-productos-by-name.use-case.ts

import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { IProductoFinancieroRepository } from '../../domain/repositories/producto-financiero.repository.interface';
import { ProductoFinanciero } from '../../domain/entities/producto-financiero.entity';

@Injectable({
    providedIn: 'root'
})
export class SearchProductosByNameUseCase {
    constructor(private readonly repository: IProductoFinancieroRepository) {}

    execute(name: string): Observable<ProductoFinanciero[]> {
        if (!name || name.trim() === '') {
            throw new Error('El nombre de búsqueda es obligatorio');
        }
        return this.repository.searchByName(name.trim());
    }
}
```

### 4️⃣ Usar en el componente

```typescript
// src/app/features/productos/components/productos-list/productos-list.component.ts

export class ProductosListComponent {
    constructor(
        // ... otros casos de uso
        private readonly searchProductosByNameUseCase: SearchProductosByNameUseCase
    ) {}

    searchByName(name: string): void {
        this.searchProductosByNameUseCase.execute(name).subscribe({
            next: (productos) => {
                this.productos = productos;
                this.applyFilters();
            },
            error: (error) => {
                console.error('Error en la búsqueda:', error);
                alert('Error al buscar productos');
            }
        });
    }
}
```

---

## 🔍 Ejemplo 3: Agregar validaciones personalizadas en la entidad

```typescript
// src/app/core/domain/entities/producto-financiero.entity.ts

export class ProductoFinanciero {
    constructor(
        public readonly id: string,
        public readonly name: string,
        public readonly description: string,
        public readonly logo: string,
        public readonly dateRelease: Date,
        public readonly dateRevision: Date
    ) {
        this.validate();
    }

    private validate(): void {
        // Validaciones existentes...

        // ✨ Nueva validación: fecha de revisión debe ser después de fecha de liberación
        if (this.dateRevision <= this.dateRelease) {
            throw new TypeError('La fecha de revisión debe ser posterior a la fecha de liberación');
        }

        // ✨ Nueva validación: nombre mínimo 3 caracteres
        if (this.name.length < 3) {
            throw new TypeError('El nombre debe tener al menos 3 caracteres');
        }

        // ✨ Nueva validación: ID con formato específico
        if (!/^[A-Z]{3}-\d{4}$/.test(this.id)) {
            throw new TypeError('El ID debe tener el formato: AAA-0000');
        }
    }

    // ✨ Nuevo método de negocio
    isExpired(): boolean {
        return new Date() > this.dateRevision;
    }

    // ✨ Nuevo método de negocio
    daysUntilRevision(): number {
        const now = new Date();
        const diff = this.dateRevision.getTime() - now.getTime();
        return Math.ceil(diff / (1000 * 60 * 60 * 24));
    }
}
```

---

## 🎨 Ejemplo 4: Agregar un nuevo servicio auxiliar

### Escenario: Servicio de exportación a Excel

```typescript
// src/app/shared/services/export.service.ts

import { Injectable } from '@angular/core';
import { ProductoFinanciero } from '../../core/domain/entities/producto-financiero.entity';

@Injectable({
    providedIn: 'root'
})
export class ExportService {
    /**
     * Exporta productos a CSV
     */
    exportToCSV(productos: ProductoFinanciero[], filename: string = 'productos.csv'): void {
        const headers = ['ID', 'Nombre', 'Descripción', 'Logo', 'Fecha Liberación', 'Fecha Revisión'];
        const rows = productos.map(p => [
            p.id,
            p.name,
            p.description,
            p.logo,
            p.dateRelease.toISOString(),
            p.dateRevision.toISOString()
        ]);

        const csvContent = [
            headers.join(','),
            ...rows.map(row => row.join(','))
        ].join('\n');

        this.downloadFile(csvContent, filename, 'text/csv');
    }

    /**
     * Exporta productos a JSON
     */
    exportToJSON(productos: ProductoFinanciero[], filename: string = 'productos.json'): void {
        const jsonContent = JSON.stringify(
            productos.map(p => p.toPlainObject()),
            null,
            2
        );
        this.downloadFile(jsonContent, filename, 'application/json');
    }

    private downloadFile(content: string, filename: string, type: string): void {
        const blob = new Blob([content], { type });
        const url = window.URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        window.URL.revokeObjectURL(url);
    }
}
```

### Usar en el componente

```typescript
export class ProductosListComponent {
    constructor(
        // ... otros servicios
        private readonly exportService: ExportService
    ) {}

    exportProductsToCSV(): void {
        this.exportService.exportToCSV(this.productos, 'productos.csv');
    }

    exportProductsToJSON(): void {
        this.exportService.exportToJSON(this.productos, 'productos.json');
    }
}
```

---

## 🧩 Ejemplo 5: Agregar interceptors HTTP

```typescript
// src/app/core/infrastructure/interceptors/auth.interceptor.ts

import { Injectable } from '@angular/core';
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable()
export class AuthInterceptor implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        // Agregar token de autenticación
        const token = localStorage.getItem('auth_token');
        
        if (token) {
            const cloned = req.clone({
                headers: req.headers.set('Authorization', `Bearer ${token}`)
            });
            return next.handle(cloned);
        }
        
        return next.handle(req);
    }
}
```

### Registrar en app.config.ts

```typescript
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './core/infrastructure/interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])
    ),
    // ... otros providers
  ]
};
```

---

## 🎯 Ejemplo 6: Agregar caché al repositorio

```typescript
// src/app/core/infrastructure/repositories/producto-financiero-cached.repository.ts

import { Injectable } from '@angular/core';
import { Observable, of } from 'rxjs';
import { tap } from 'rxjs/operators';
import { IProductoFinancieroRepository } from '../../domain/repositories/producto-financiero.repository.interface';
import { ProductoFinanciero } from '../../domain/entities/producto-financiero.entity';
import { ProductoFinancieroHttpRepository } from './producto-financiero-http.repository';

@Injectable({
    providedIn: 'root'
})
export class ProductoFinancieroCachedRepository implements IProductoFinancieroRepository {
    private cache: Map<string, ProductoFinanciero[]> = new Map();
    private cacheExpiry: number = 5 * 60 * 1000; // 5 minutos

    constructor(private readonly httpRepository: ProductoFinancieroHttpRepository) {}

    getAll(): Observable<ProductoFinanciero[]> {
        const cached = this.cache.get('all');
        if (cached) {
            return of(cached);
        }

        return this.httpRepository.getAll().pipe(
            tap(productos => this.cache.set('all', productos))
        );
    }

    getById(id: string): Observable<ProductoFinanciero> {
        return this.httpRepository.getById(id);
    }

    create(producto: ProductoFinanciero): Observable<void> {
        this.cache.clear(); // Limpiar caché después de crear
        return this.httpRepository.create(producto);
    }

    update(producto: ProductoFinanciero): Observable<void> {
        this.cache.clear(); // Limpiar caché después de actualizar
        return this.httpRepository.update(producto);
    }

    delete(id: string): Observable<void> {
        this.cache.clear(); // Limpiar caché después de eliminar
        return this.httpRepository.delete(id);
    }
}
```

---

## 🧪 Ejemplo 7: Testing de casos de uso

```typescript
// src/app/core/application/use-cases/create-producto.use-case.spec.ts

import { TestBed } from '@angular/core/testing';
import { of, throwError } from 'rxjs';
import { CreateProductoUseCase } from './create-producto.use-case';
import { IProductoFinancieroRepository } from '../../domain/repositories/producto-financiero.repository.interface';
import { ProductoFinanciero } from '../../domain/entities/producto-financiero.entity';

describe('CreateProductoUseCase', () => {
    let useCase: CreateProductoUseCase;
    let mockRepository: jasmine.SpyObj<IProductoFinancieroRepository>;

    beforeEach(() => {
        mockRepository = jasmine.createSpyObj('IProductoFinancieroRepository', ['create']);
        TestBed.configureTestingModule({
            providers: [
                CreateProductoUseCase,
                { provide: IProductoFinancieroRepository, useValue: mockRepository }
            ]
        });
        useCase = TestBed.inject(CreateProductoUseCase);
    });

    it('debe crear un producto exitosamente', (done) => {
        const producto = new ProductoFinanciero(
            'test-id',
            'Test Product',
            'Test Description',
            'logo.png',
            new Date('2024-01-01'),
            new Date('2024-12-31')
        );

        mockRepository.create.and.returnValue(of(undefined));

        useCase.execute(producto).subscribe({
            next: () => {
                expect(mockRepository.create).toHaveBeenCalledWith(producto);
                done();
            }
        });
    });

    it('debe lanzar error si el producto es null', () => {
        expect(() => useCase.execute(null as any)).toThrowError('El producto es obligatorio');
    });

    it('debe manejar errores del repositorio', (done) => {
        const producto = new ProductoFinanciero(
            'test-id',
            'Test Product',
            'Test Description',
            'logo.png',
            new Date('2024-01-01'),
            new Date('2024-12-31')
        );

        mockRepository.create.and.returnValue(throwError(() => new Error('Error de red')));

        useCase.execute(producto).subscribe({
            error: (error) => {
                expect(error.message).toBe('Error de red');
                done();
            }
        });
    });
});
```

---

## 🎨 Ejemplo 8: Agregar un nuevo módulo (Clientes)

### 1️⃣ Entidad de dominio

```typescript
// src/app/core/domain/entities/cliente.entity.ts

export class Cliente {
    constructor(
        public readonly id: string,
        public readonly nombre: string,
        public readonly email: string,
        public readonly telefono: string
    ) {
        this.validate();
    }

    private validate(): void {
        if (!this.id || this.id.trim() === '') {
            throw new Error('El ID del cliente es obligatorio');
        }
        // ... más validaciones
    }

    static fromPlainObject(data: any): Cliente {
        return new Cliente(data.id, data.nombre, data.email, data.telefono);
    }
}
```

### 2️⃣ Interfaz del repositorio

```typescript
// src/app/core/domain/repositories/cliente.repository.interface.ts

import { Observable } from 'rxjs';
import { Cliente } from '../entities/cliente.entity';

export abstract class IClienteRepository {
    abstract getAll(): Observable<Cliente[]>;
    abstract getById(id: string): Observable<Cliente>;
    abstract create(cliente: Cliente): Observable<void>;
    abstract update(cliente: Cliente): Observable<void>;
    abstract delete(id: string): Observable<void>;
}
```

### 3️⃣ Casos de uso

```typescript
// src/app/core/application/use-cases/get-all-clientes.use-case.ts

import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { IClienteRepository } from '../../domain/repositories/cliente.repository.interface';
import { Cliente } from '../../domain/entities/cliente.entity';

@Injectable({ providedIn: 'root' })
export class GetAllClientesUseCase {
    constructor(private readonly repository: IClienteRepository) {}
    
    execute(): Observable<Cliente[]> {
        return this.repository.getAll();
    }
}
```

### 4️⃣ Implementación del repositorio

```typescript
// src/app/core/infrastructure/repositories/cliente-http.repository.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';
import { IClienteRepository } from '../../domain/repositories/cliente.repository.interface';
import { Cliente } from '../../domain/entities/cliente.entity';

@Injectable({ providedIn: 'root' })
export class ClienteHttpRepository implements IClienteRepository {
    private readonly apiUrl = 'http://localhost:3002/clientes';

    constructor(private readonly http: HttpClient) {}

    getAll(): Observable<Cliente[]> {
        return this.http.get<any[]>(this.apiUrl).pipe(
            map(data => data.map(item => Cliente.fromPlainObject(item)))
        );
    }

    // ... otros métodos
}
```

### 5️⃣ Configurar en app.config.ts

```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    // ... otros providers
    {
      provide: IClienteRepository,
      useClass: ClienteHttpRepository
    }
  ]
};
```

---

## ✅ Principios a seguir al extender

1. **Single Responsibility**: Cada nueva clase debe tener una sola responsabilidad
2. **Open/Closed**: Extender funcionalidad sin modificar código existente
3. **Liskov Substitution**: Las implementaciones deben ser intercambiables
4. **Interface Segregation**: Crear interfaces específicas y cohesivas
5. **Dependency Inversion**: Depender siempre de abstracciones

---

## 🎓 Resumen

Esta arquitectura es **altamente extensible** porque:
- ✅ Las capas están bien definidas
- ✅ Las dependencias están invertidas
- ✅ Cada componente tiene una responsabilidad clara
- ✅ Es fácil agregar funcionalidad sin romper lo existente
- ✅ El testing es simple gracias a la inyección de dependencias

**¡Feliz codificación! 🚀**
