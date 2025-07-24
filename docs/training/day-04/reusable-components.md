# 🧩 Componentes Reutilizables para CRUDs

**Duración:** 1.5 horas (2:30 PM - 4:00 PM)  
**Objetivo:** Crear biblioteca de componentes reutilizables que aceleren el desarrollo de interfaces CRUD

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor será capaz de:

✅ **Crear componentes de tabla** con paginación, ordenamiento y filtros  
✅ **Desarrollar formularios dinámicos** con validación integrada  
✅ **Implementar modales** reutilizables para acciones CRUD  
✅ **Construir componentes de búsqueda** con debouncing automático  
✅ **Optimizar renders** con React.memo y useMemo

---

## 🧩 **Paso 1: Componentes Base UI (15 minutos)**

Comenzamos con componentes básicos que servirán como building blocks.

### **📄 `src/components/ui/Button.jsx`:**

```jsx
import { forwardRef } from 'react';
import clsx from 'clsx';

const Button = forwardRef(
  (
    {
      children,
      variant = 'primary',
      size = 'md',
      loading = false,
      disabled = false,
      className,
      ...props
    },
    ref
  ) => {
    const baseClasses =
      'inline-flex items-center justify-center font-medium rounded-lg transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2';

    const variants = {
      primary:
        'bg-primary-500 hover:bg-primary-600 text-white focus:ring-primary-500',
      secondary:
        'bg-gray-200 hover:bg-gray-300 text-gray-800 focus:ring-gray-500',
      danger: 'bg-red-500 hover:bg-red-600 text-white focus:ring-red-500',
      success:
        'bg-green-500 hover:bg-green-600 text-white focus:ring-green-500',
      outline:
        'border-2 border-primary-500 text-primary-500 hover:bg-primary-50 focus:ring-primary-500',
    };

    const sizes = {
      sm: 'px-3 py-1.5 text-sm',
      md: 'px-4 py-2 text-sm',
      lg: 'px-6 py-3 text-base',
    };

    const isDisabled = disabled || loading;

    return (
      <button
        ref={ref}
        disabled={isDisabled}
        className={clsx(
          baseClasses,
          variants[variant],
          sizes[size],
          isDisabled && 'opacity-50 cursor-not-allowed',
          className
        )}
        {...props}>
        {loading && (
          <svg
            className="animate-spin -ml-1 mr-2 h-4 w-4"
            fill="none"
            viewBox="0 0 24 24">
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="m4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
            />
          </svg>
        )}
        {children}
      </button>
    );
  }
);

Button.displayName = 'Button';

export default Button;
```

### **📄 `src/components/ui/Input.jsx`:**

```jsx
import { forwardRef } from 'react';
import clsx from 'clsx';

const Input = forwardRef(
  (
    { label, error, helperText, className, containerClassName, ...props },
    ref
  ) => {
    const hasError = !!error;

    return (
      <div className={clsx('space-y-1', containerClassName)}>
        {label && (
          <label className="block text-sm font-medium text-gray-700">
            {label}
            {props.required && <span className="text-red-500 ml-1">*</span>}
          </label>
        )}

        <input
          ref={ref}
          className={clsx(
            'block w-full rounded-md shadow-sm transition-colors duration-200',
            hasError
              ? 'border-red-300 text-red-900 placeholder-red-300 focus:border-red-500 focus:ring-red-500'
              : 'border-gray-300 focus:border-primary-500 focus:ring-primary-500',
            className
          )}
          {...props}
        />

        {error && <p className="text-sm text-red-600">{error}</p>}

        {helperText && !error && (
          <p className="text-sm text-gray-500">{helperText}</p>
        )}
      </div>
    );
  }
);

Input.displayName = 'Input';

export default Input;
```

### **📄 `src/components/ui/Modal.jsx`:**

```jsx
import { Fragment } from 'react';
import { Dialog, Transition } from '@headlessui/react';
import { XMarkIcon } from '@heroicons/react/24/outline';
import Button from './Button';

const Modal = ({
  isOpen,
  onClose,
  title,
  children,
  footer,
  size = 'md',
  showCloseButton = true,
}) => {
  const sizes = {
    sm: 'max-w-md',
    md: 'max-w-lg',
    lg: 'max-w-2xl',
    xl: 'max-w-4xl',
  };

  return (
    <Transition
      appear
      show={isOpen}
      as={Fragment}>
      <Dialog
        as="div"
        className="relative z-50"
        onClose={onClose}>
        <Transition.Child
          as={Fragment}
          enter="ease-out duration-300"
          enterFrom="opacity-0"
          enterTo="opacity-100"
          leave="ease-in duration-200"
          leaveFrom="opacity-100"
          leaveTo="opacity-0">
          <div className="fixed inset-0 bg-black bg-opacity-25" />
        </Transition.Child>

        <div className="fixed inset-0 overflow-y-auto">
          <div className="flex min-h-full items-center justify-center p-4 text-center">
            <Transition.Child
              as={Fragment}
              enter="ease-out duration-300"
              enterFrom="opacity-0 scale-95"
              enterTo="opacity-100 scale-100"
              leave="ease-in duration-200"
              leaveFrom="opacity-100 scale-100"
              leaveTo="opacity-0 scale-95">
              <Dialog.Panel
                className={clsx(
                  'w-full transform overflow-hidden rounded-2xl bg-white p-6 text-left align-middle shadow-xl transition-all',
                  sizes[size]
                )}>
                <div className="flex items-center justify-between mb-4">
                  <Dialog.Title className="text-lg font-medium text-gray-900">
                    {title}
                  </Dialog.Title>

                  {showCloseButton && (
                    <button
                      type="button"
                      className="rounded-md text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-primary-500"
                      onClick={onClose}>
                      <XMarkIcon className="h-6 w-6" />
                    </button>
                  )}
                </div>

                <div className="mb-6">{children}</div>

                {footer && (
                  <div className="flex justify-end space-x-2">{footer}</div>
                )}
              </Dialog.Panel>
            </Transition.Child>
          </div>
        </div>
      </Dialog>
    </Transition>
  );
};

export default Modal;
```

---

## 🧩 **Paso 2: Componente DataTable Avanzado (25 minutos)**

Una tabla reutilizable con todas las funcionalidades necesarias para WorldSkills.

**📄 `src/components/tables/DataTable.jsx`:**

```jsx
import { useState, useMemo } from 'react';
import { ChevronUpIcon, ChevronDownIcon } from '@heroicons/react/20/solid';
import Button from '../ui/Button';
import clsx from 'clsx';

const DataTable = ({
  data = [],
  columns = [],
  loading = false,
  emptyMessage = 'No hay datos disponibles',
  onRowClick,
  sortable = true,
  pagination = true,
  pageSize = 10,
  actions,
  className,
}) => {
  const [sortField, setSortField] = useState('');
  const [sortDirection, setSortDirection] = useState('asc');
  const [currentPage, setCurrentPage] = useState(1);

  // Ordenamiento
  const sortedData = useMemo(() => {
    if (!sortField) return data;

    return [...data].sort((a, b) => {
      const aValue = a[sortField];
      const bValue = b[sortField];

      if (aValue < bValue) return sortDirection === 'asc' ? -1 : 1;
      if (aValue > bValue) return sortDirection === 'asc' ? 1 : -1;
      return 0;
    });
  }, [data, sortField, sortDirection]);

  // Paginación
  const paginatedData = useMemo(() => {
    if (!pagination) return sortedData;

    const startIndex = (currentPage - 1) * pageSize;
    return sortedData.slice(startIndex, startIndex + pageSize);
  }, [sortedData, currentPage, pageSize, pagination]);

  // Información de paginación
  const paginationInfo = useMemo(() => {
    const totalItems = sortedData.length;
    const totalPages = Math.ceil(totalItems / pageSize);
    const startItem = (currentPage - 1) * pageSize + 1;
    const endItem = Math.min(currentPage * pageSize, totalItems);

    return { totalItems, totalPages, startItem, endItem };
  }, [sortedData.length, currentPage, pageSize]);

  const handleSort = (field) => {
    if (!sortable) return;

    if (sortField === field) {
      setSortDirection(sortDirection === 'asc' ? 'desc' : 'asc');
    } else {
      setSortField(field);
      setSortDirection('asc');
    }
  };

  const SortIcon = ({ field }) => {
    if (sortField !== field) return null;

    return sortDirection === 'asc' ? (
      <ChevronUpIcon className="w-4 h-4" />
    ) : (
      <ChevronDownIcon className="w-4 h-4" />
    );
  };

  if (loading) {
    return (
      <div className="text-center py-8">
        <div className="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-primary-500"></div>
        <p className="mt-2 text-sm text-gray-500">Cargando...</p>
      </div>
    );
  }

  return (
    <div
      className={clsx('bg-white shadow rounded-lg overflow-hidden', className)}>
      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              {columns.map((column) => (
                <th
                  key={column.key}
                  scope="col"
                  className={clsx(
                    'px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider',
                    sortable &&
                      column.sortable !== false &&
                      'cursor-pointer hover:bg-gray-100',
                    column.className
                  )}
                  onClick={() =>
                    column.sortable !== false && handleSort(column.key)
                  }>
                  <div className="flex items-center space-x-1">
                    <span>{column.title}</span>
                    <SortIcon field={column.key} />
                  </div>
                </th>
              ))}
              {actions && (
                <th
                  scope="col"
                  className="relative px-6 py-3">
                  <span className="sr-only">Acciones</span>
                </th>
              )}
            </tr>
          </thead>

          <tbody className="bg-white divide-y divide-gray-200">
            {paginatedData.length === 0 ? (
              <tr>
                <td
                  colSpan={columns.length + (actions ? 1 : 0)}
                  className="px-6 py-8 text-center text-sm text-gray-500">
                  {emptyMessage}
                </td>
              </tr>
            ) : (
              paginatedData.map((row, index) => (
                <tr
                  key={row.id || index}
                  className={clsx(
                    'hover:bg-gray-50 transition-colors duration-150',
                    onRowClick && 'cursor-pointer'
                  )}
                  onClick={() => onRowClick?.(row)}>
                  {columns.map((column) => (
                    <td
                      key={column.key}
                      className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                      {column.render
                        ? column.render(row[column.key], row)
                        : row[column.key]}
                    </td>
                  ))}

                  {actions && (
                    <td className="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                      <div className="flex justify-end space-x-2">
                        {actions(row)}
                      </div>
                    </td>
                  )}
                </tr>
              ))
            )}
          </tbody>
        </table>
      </div>

      {pagination && paginationInfo.totalPages > 1 && (
        <div className="bg-white px-4 py-3 flex items-center justify-between border-t border-gray-200 sm:px-6">
          <div className="flex-1 flex justify-between sm:hidden">
            <Button
              variant="outline"
              size="sm"
              onClick={() => setCurrentPage(currentPage - 1)}
              disabled={currentPage === 1}>
              Anterior
            </Button>
            <Button
              variant="outline"
              size="sm"
              onClick={() => setCurrentPage(currentPage + 1)}
              disabled={currentPage === paginationInfo.totalPages}>
              Siguiente
            </Button>
          </div>

          <div className="hidden sm:flex-1 sm:flex sm:items-center sm:justify-between">
            <div>
              <p className="text-sm text-gray-700">
                Mostrando{' '}
                <span className="font-medium">{paginationInfo.startItem}</span>{' '}
                a <span className="font-medium">{paginationInfo.endItem}</span>{' '}
                de{' '}
                <span className="font-medium">{paginationInfo.totalItems}</span>{' '}
                resultados
              </p>
            </div>

            <div>
              <nav className="relative z-0 inline-flex rounded-md shadow-sm -space-x-px">
                <button
                  onClick={() => setCurrentPage(currentPage - 1)}
                  disabled={currentPage === 1}
                  className="relative inline-flex items-center px-2 py-2 rounded-l-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed">
                  Anterior
                </button>

                {Array.from(
                  { length: paginationInfo.totalPages },
                  (_, i) => i + 1
                ).map((page) => (
                  <button
                    key={page}
                    onClick={() => setCurrentPage(page)}
                    className={clsx(
                      'relative inline-flex items-center px-4 py-2 border text-sm font-medium',
                      page === currentPage
                        ? 'z-10 bg-primary-50 border-primary-500 text-primary-600'
                        : 'bg-white border-gray-300 text-gray-500 hover:bg-gray-50'
                    )}>
                    {page}
                  </button>
                ))}

                <button
                  onClick={() => setCurrentPage(currentPage + 1)}
                  disabled={currentPage === paginationInfo.totalPages}
                  className="relative inline-flex items-center px-2 py-2 rounded-r-md border border-gray-300 bg-white text-sm font-medium text-gray-500 hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed">
                  Siguiente
                </button>
              </nav>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default DataTable;
```

---

## 🧩 **Paso 3: Formulario Dinámico (20 minutos)**

Un componente que genere formularios automáticamente basado en configuración.

**📄 `src/components/forms/DynamicForm.jsx`:**

```jsx
import { useEffect } from 'react';
import useForm from '../../hooks/useForm';
import Input from '../ui/Input';
import Button from '../ui/Button';

const DynamicForm = ({
  fields,
  initialValues = {},
  validationRules = {},
  onSubmit,
  loading = false,
  submitText = 'Guardar',
  cancelText = 'Cancelar',
  onCancel,
  className,
}) => {
  const { values, errors, isValid, getFieldProps, validate, reset } = useForm(
    initialValues,
    validationRules
  );

  useEffect(() => {
    if (Object.keys(initialValues).length > 0) {
      reset();
    }
  }, [initialValues, reset]);

  const handleSubmit = async (e) => {
    e.preventDefault();

    if (!validate()) {
      return;
    }

    try {
      await onSubmit(values);
    } catch (error) {
      console.error('Error en formulario:', error);
    }
  };

  const renderField = (field) => {
    const fieldProps = getFieldProps(field.name);

    switch (field.type) {
      case 'select':
        return (
          <div
            key={field.name}
            className="space-y-1">
            <label className="block text-sm font-medium text-gray-700">
              {field.label}
              {field.required && <span className="text-red-500 ml-1">*</span>}
            </label>
            <select
              {...fieldProps}
              className="block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
              required={field.required}>
              <option value="">Seleccionar...</option>
              {field.options?.map((option) => (
                <option
                  key={option.value}
                  value={option.value}>
                  {option.label}
                </option>
              ))}
            </select>
            {fieldProps.error && (
              <p className="text-sm text-red-600">{fieldProps.error}</p>
            )}
          </div>
        );

      case 'textarea':
        return (
          <div
            key={field.name}
            className="space-y-1">
            <label className="block text-sm font-medium text-gray-700">
              {field.label}
              {field.required && <span className="text-red-500 ml-1">*</span>}
            </label>
            <textarea
              {...fieldProps}
              rows={field.rows || 3}
              className="block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500"
              placeholder={field.placeholder}
              required={field.required}
            />
            {fieldProps.error && (
              <p className="text-sm text-red-600">{fieldProps.error}</p>
            )}
          </div>
        );

      case 'checkbox':
        return (
          <div
            key={field.name}
            className="flex items-center">
            <input
              type="checkbox"
              {...fieldProps}
              checked={fieldProps.value || false}
              onChange={(e) =>
                fieldProps.onChange({ target: { value: e.target.checked } })
              }
              className="h-4 w-4 text-primary-600 focus:ring-primary-500 border-gray-300 rounded"
            />
            <label className="ml-2 block text-sm text-gray-900">
              {field.label}
            </label>
          </div>
        );

      default:
        return (
          <Input
            key={field.name}
            label={field.label}
            type={field.type || 'text'}
            placeholder={field.placeholder}
            required={field.required}
            error={fieldProps.error}
            {...fieldProps}
          />
        );
    }
  };

  return (
    <form
      onSubmit={handleSubmit}
      className={className}>
      <div className="space-y-4">{fields.map(renderField)}</div>

      <div className="mt-6 flex justify-end space-x-3">
        {onCancel && (
          <Button
            type="button"
            variant="secondary"
            onClick={onCancel}
            disabled={loading}>
            {cancelText}
          </Button>
        )}

        <Button
          type="submit"
          loading={loading}
          disabled={!isValid}>
          {submitText}
        </Button>
      </div>
    </form>
  );
};

export default DynamicForm;
```

---

## 🧩 **Paso 4: Componente de Búsqueda (15 minutos)**

Búsqueda con debouncing y filtros avanzados.

**📄 `src/components/search/SearchFilter.jsx`:**

```jsx
import { useState } from 'react';
import { MagnifyingGlassIcon, FunnelIcon } from '@heroicons/react/24/outline';
import useDebounce from '../../hooks/useDebounce';
import Input from '../ui/Input';
import Button from '../ui/Button';

const SearchFilter = ({
  onSearch,
  onFilter,
  filters = [],
  placeholder = 'Buscar...',
  debounceDelay = 300,
  showFilters = true,
}) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [filterValues, setFilterValues] = useState({});
  const [showFilterPanel, setShowFilterPanel] = useState(false);

  const debouncedSearchTerm = useDebounce(searchTerm, debounceDelay);

  // Ejecutar búsqueda cuando cambie el término debounced
  useEffect(() => {
    onSearch?.(debouncedSearchTerm);
  }, [debouncedSearchTerm, onSearch]);

  // Aplicar filtros
  const handleApplyFilters = () => {
    onFilter?.(filterValues);
    setShowFilterPanel(false);
  };

  // Limpiar filtros
  const handleClearFilters = () => {
    setFilterValues({});
    onFilter?.({});
  };

  const hasActiveFilters = Object.values(filterValues).some(
    (value) => value !== '' && value !== null && value !== undefined
  );

  return (
    <div className="space-y-4">
      <div className="flex items-center space-x-4">
        {/* Búsqueda */}
        <div className="flex-1 relative">
          <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <MagnifyingGlassIcon className="h-5 w-5 text-gray-400" />
          </div>
          <input
            type="text"
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
            placeholder={placeholder}
            className="block w-full pl-10 pr-3 py-2 border border-gray-300 rounded-md leading-5 bg-white placeholder-gray-500 focus:outline-none focus:placeholder-gray-400 focus:ring-1 focus:ring-primary-500 focus:border-primary-500"
          />
        </div>

        {/* Botón de filtros */}
        {showFilters && filters.length > 0 && (
          <Button
            type="button"
            variant="outline"
            onClick={() => setShowFilterPanel(!showFilterPanel)}
            className={
              hasActiveFilters ? 'border-primary-500 text-primary-600' : ''
            }>
            <FunnelIcon className="h-5 w-5 mr-2" />
            Filtros
            {hasActiveFilters && (
              <span className="ml-2 bg-primary-500 text-white text-xs rounded-full h-5 w-5 flex items-center justify-center">
                {Object.values(filterValues).filter((v) => v).length}
              </span>
            )}
          </Button>
        )}
      </div>

      {/* Panel de filtros */}
      {showFilterPanel && (
        <div className="bg-gray-50 border border-gray-200 rounded-lg p-4">
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {filters.map((filter) => (
              <div key={filter.key}>
                {filter.type === 'select' ? (
                  <div>
                    <label className="block text-sm font-medium text-gray-700 mb-1">
                      {filter.label}
                    </label>
                    <select
                      value={filterValues[filter.key] || ''}
                      onChange={(e) =>
                        setFilterValues((prev) => ({
                          ...prev,
                          [filter.key]: e.target.value,
                        }))
                      }
                      className="block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500 text-sm">
                      <option value="">Todos</option>
                      {filter.options?.map((option) => (
                        <option
                          key={option.value}
                          value={option.value}>
                          {option.label}
                        </option>
                      ))}
                    </select>
                  </div>
                ) : (
                  <Input
                    label={filter.label}
                    type={filter.type || 'text'}
                    value={filterValues[filter.key] || ''}
                    onChange={(e) =>
                      setFilterValues((prev) => ({
                        ...prev,
                        [filter.key]: e.target.value,
                      }))
                    }
                    placeholder={filter.placeholder}
                  />
                )}
              </div>
            ))}
          </div>

          <div className="mt-4 flex justify-end space-x-2">
            <Button
              type="button"
              variant="secondary"
              size="sm"
              onClick={handleClearFilters}>
              Limpiar
            </Button>
            <Button
              type="button"
              size="sm"
              onClick={handleApplyFilters}>
              Aplicar Filtros
            </Button>
          </div>
        </div>
      )}
    </div>
  );
};

export default SearchFilter;
```

---

## 🧩 **Paso 5: CRUD Component Template (15 minutos)**

Un template completo que combine todos los componentes anteriores.

**📄 `src/components/crud/CrudTemplate.jsx`:**

```jsx
import { useState } from 'react';
import { PlusIcon, PencilIcon, TrashIcon } from '@heroicons/react/24/outline';
import DataTable from '../tables/DataTable';
import SearchFilter from '../search/SearchFilter';
import DynamicForm from '../forms/DynamicForm';
import Modal from '../ui/Modal';
import Button from '../ui/Button';

const CrudTemplate = ({
  title,
  data,
  loading,
  columns,
  searchFilters = [],
  formFields = [],
  validationRules = {},
  onSearch,
  onFilter,
  onCreate,
  onUpdate,
  onDelete,
  createPermission = true,
  updatePermission = true,
  deletePermission = true,
}) => {
  const [showCreateModal, setShowCreateModal] = useState(false);
  const [showEditModal, setShowEditModal] = useState(false);
  const [selectedItem, setSelectedItem] = useState(null);
  const [formLoading, setFormLoading] = useState(false);

  const handleCreate = async (data) => {
    setFormLoading(true);
    try {
      await onCreate(data);
      setShowCreateModal(false);
    } finally {
      setFormLoading(false);
    }
  };

  const handleUpdate = async (data) => {
    setFormLoading(true);
    try {
      await onUpdate(selectedItem.id, data);
      setShowEditModal(false);
      setSelectedItem(null);
    } finally {
      setFormLoading(false);
    }
  };

  const handleEdit = (item) => {
    setSelectedItem(item);
    setShowEditModal(true);
  };

  const handleDelete = async (item) => {
    if (window.confirm('¿Estás seguro de que deseas eliminar este elemento?')) {
      await onDelete(item.id);
    }
  };

  const tableActions = (row) => (
    <>
      {updatePermission && (
        <Button
          size="sm"
          variant="outline"
          onClick={() => handleEdit(row)}>
          <PencilIcon className="h-4 w-4" />
        </Button>
      )}
      {deletePermission && (
        <Button
          size="sm"
          variant="danger"
          onClick={() => handleDelete(row)}>
          <TrashIcon className="h-4 w-4" />
        </Button>
      )}
    </>
  );

  return (
    <div className="space-y-6">
      {/* Header */}
      <div className="flex justify-between items-center">
        <h1 className="text-2xl font-bold text-gray-900">{title}</h1>
        {createPermission && (
          <Button onClick={() => setShowCreateModal(true)}>
            <PlusIcon className="h-5 w-5 mr-2" />
            Crear Nuevo
          </Button>
        )}
      </div>

      {/* Búsqueda y Filtros */}
      <SearchFilter
        onSearch={onSearch}
        onFilter={onFilter}
        filters={searchFilters}
      />

      {/* Tabla */}
      <DataTable
        data={data}
        columns={columns}
        loading={loading}
        actions={tableActions}
      />

      {/* Modal de Creación */}
      <Modal
        isOpen={showCreateModal}
        onClose={() => setShowCreateModal(false)}
        title={`Crear ${title.slice(0, -1)}`}>
        <DynamicForm
          fields={formFields}
          validationRules={validationRules}
          onSubmit={handleCreate}
          onCancel={() => setShowCreateModal(false)}
          loading={formLoading}
        />
      </Modal>

      {/* Modal de Edición */}
      <Modal
        isOpen={showEditModal}
        onClose={() => {
          setShowEditModal(false);
          setSelectedItem(null);
        }}
        title={`Editar ${title.slice(0, -1)}`}>
        <DynamicForm
          fields={formFields}
          initialValues={selectedItem || {}}
          validationRules={validationRules}
          onSubmit={handleUpdate}
          onCancel={() => {
            setShowEditModal(false);
            setSelectedItem(null);
          }}
          loading={formLoading}
        />
      </Modal>
    </div>
  );
};

export default CrudTemplate;
```

---

## 🧪 **Ejemplo de Uso: Página de Países (5 minutos)**

Demostrar cómo usar todos estos componentes juntos.

**📄 `src/pages/Countries/Countries.jsx`:**

```jsx
import { useCountries, useCountryOperations } from '../../hooks/useCountries';
import CrudTemplate from '../../components/crud/CrudTemplate';

const Countries = () => {
  const { countries, loading, refetch } = useCountries();
  const { create, update, delete: deleteCountry } = useCountryOperations();

  const columns = [
    {
      key: 'name',
      title: 'Nombre',
      sortable: true,
    },
    {
      key: 'code',
      title: 'Código',
      sortable: true,
    },
    {
      key: 'teams_count',
      title: 'Equipos',
      render: (value) => value || 0,
    },
  ];

  const formFields = [
    {
      name: 'name',
      label: 'Nombre del País',
      type: 'text',
      required: true,
      placeholder: 'Ej: Colombia',
    },
    {
      name: 'code',
      label: 'Código ISO',
      type: 'text',
      required: true,
      placeholder: 'Ej: CO',
    },
  ];

  const validationRules = {
    name: [
      { required: true, message: 'El nombre es requerido' },
      { minLength: 2, message: 'El nombre debe tener al menos 2 caracteres' },
    ],
    code: [
      { required: true, message: 'El código es requerido' },
      {
        pattern: /^[A-Z]{2}$/,
        message: 'El código debe tener 2 letras mayúsculas',
      },
    ],
  };

  const handleCreate = async (data) => {
    await create.execute(data);
    refetch();
  };

  const handleUpdate = async (id, data) => {
    await update.execute(id, data);
    refetch();
  };

  const handleDelete = async (id) => {
    await deleteCountry.execute(id);
    refetch();
  };

  return (
    <CrudTemplate
      title="Países"
      data={countries}
      loading={loading}
      columns={columns}
      formFields={formFields}
      validationRules={validationRules}
      onCreate={handleCreate}
      onUpdate={handleUpdate}
      onDelete={handleDelete}
    />
  );
};

export default Countries;
```

---

## 💡 **Tips de Optimización**

### **React.memo para Componentes Pesados:**

```jsx
import { memo } from 'react';

const DataTable = memo(
  ({ data, columns, ...props }) => {
    // ... componente
  },
  (prevProps, nextProps) => {
    // Custom comparison para optimizar re-renders
    return (
      prevProps.data === nextProps.data &&
      prevProps.loading === nextProps.loading
    );
  }
);
```

### **useMemo para Cálculos Costosos:**

```jsx
const processedData = useMemo(() => {
  return data.map((item) => ({
    ...item,
    displayName: `${item.name} (${item.code})`,
    status: item.active ? 'Activo' : 'Inactivo',
  }));
}, [data]);
```

---

## 🎯 **Siguientes Pasos**

Con estos componentes reutilizables, puedes:

1. **Crear cualquier CRUD** en minutos usando CrudTemplate
2. **Personalizar interfaces** específicas del dominio deportivo
3. **Mantener consistencia** visual en toda la aplicación
4. **Acelerar desarrollo** reutilizando componentes probados

¡Estos componentes son tu arsenal para desarrollo React ultrarrápido en WorldSkills! 🚀
