#  Слоты в Vue.js

## 📖 Что такое слоты?

**Слоты (Slots)** — это мощный механизм Vue.js для создания гибких и переиспользуемых компонентов. Они позволяют передавать контент из родительского компонента в дочерний, сохраняя полный контроль над разметкой.

##  Зачем нужны слоты?

- ✅ Создание переиспользуемых компонентов
- ✅ Кастомизация содержимого компонента
- ✅ Передача HTML/компонентов в дочерний компонент
- ✅ Создание компонентов-оберток (layout)
- ✅ Инверсия управления содержимым

## 📚 Виды слотов

### 1. Базовые (по умолчанию)

```vue
<!-- ChildComponent.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <slot>Заголовок по умолчанию</slot>
    </div>
    <div class="card-body">
      <slot name="body">Тело по умолчанию</slot>
    </div>
  </div>
</template>

<!-- ParentComponent.vue -->
<template>
  <ChildComponent>
    <h2>Мой кастомный заголовок</h2>
    
    <template #body>
      <p>Мой кастомный контент</p>
      <button>Кнопка</button>
    </template>
  </ChildComponent>
</template>

2. Именованные слоты

<!-- LayoutComponent.vue -->
<template>
  <div class="layout">
    <header>
      <slot name="header"></slot>
    </header>
    
    <main>
      <slot name="main"></slot>
    </main>
    
    <aside>
      <slot name="sidebar"></slot>
    </aside>
    
    <footer>
      <slot name="footer">
        <p>© 2024 Все права защищены</p>
      </slot>
    </footer>
  </div>
</template>

<!-- Использование -->
<template>
  <LayoutComponent>
    <template #header>
      <nav>Меню навигации</nav>
    </template>
    
    <template #main>
      <article>Основной контент</article>
    </template>
    
    <template #sidebar>
      <aside>Боковая панель</aside>
    </template>
    
    <!-- #footer использует содержимое по умолчанию -->
  </LayoutComponent>
</template>
3. Слоты с динамическими именами

<template>
  <component :is="currentComponent">
    <template v-for="slotName in slotsList" #[slotName]="slotProps">
      <div class="dynamic-slot">
        <slot :name="slotName" v-bind="slotProps"></slot>
      </div>
    </template>
  </component>
</template>

<script setup>
const slotsList = ['header', 'content', 'footer']
</script>
4. Скапированные слоты (Scoped Slots)

<!-- DataTable.vue -->
<template>
  <div class="data-table">
    <table>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.key">
            {{ column.title }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in data" :key="row.id">
          <td v-for="column in columns" :key="column.key">
            <slot :name="`column-${column.key}`" :row="row" :value="row[column.key]">
              <!-- Значение по умолчанию -->
              {{ row[column.key] }}
            </slot>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
const props = defineProps({
  data: Array,
  columns: Array
})
</script>

<!-- Использование DataTable -->
<template>
  <DataTable :data="users" :columns="columns">
    <!-- Кастомный шаблон для колонки name -->
    <template #column-name="{ row, value }">
      <strong>{{ value }}</strong>
      <span v-if="row.isVIP">👑</span>
    </template>
    
    <!-- Кастомный шаблон для колонки actions -->
    <template #column-actions="{ row }">
      <button @click="editUser(row)">Редактировать</button>
      <button @click="deleteUser(row)">Удалить</button>
    </template>
  </DataTable>
</template>

<script setup>
const users = ref([
  { id: 1, name: 'Анна', email: 'anna@example.com', isVIP: true },
  { id: 2, name: 'Петр', email: 'petr@example.com', isVIP: false }
])

const columns = [
  { key: 'name', title: 'Имя' },
  { key: 'email', title: 'Email' },
  { key: 'actions', title: 'Действия' }
]
</script>
 Реальные примеры использования
Пример 1: Модальное окно

<!-- Modal.vue -->
<template>
  <div v-if="isOpen" class="modal-overlay" @click="close">
    <div class="modal-content" @click.stop>
      <div class="modal-header">
        <slot name="header">
          <h3>Модальное окно</h3>
        </slot>
        <button class="close-btn" @click="close">×</button>
      </div>
      
      <div class="modal-body">
        <slot name="body">
          <p>Содержимое по умолчанию</p>
        </slot>
      </div>
      
      <div class="modal-footer">
        <slot name="footer">
          <button @click="close">Закрыть</button>
          <button @click="confirm">Подтвердить</button>
        </slot>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  modelValue: Boolean
})

const emit = defineEmits(['update:modelValue', 'confirm'])

const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const close = () => {
  isOpen.value = false
}

const confirm = () => {
  emit('confirm')
  close()
}
</script>

<!-- Использование -->
<template>
  <button @click="showModal = true">Открыть модалку</button>
  
  <Modal v-model="showModal" @confirm="handleConfirm">
    <template #header>
      <h2>Удаление пользователя</h2>
    </template>
    
    <template #body>
      <p>Вы уверены, что хотите удалить пользователя <strong>{{ userName }}</strong>?</p>
      <p>Это действие необратимо!</p>
    </template>
    
    <template #footer>
      <button class="cancel" @click="showModal = false">Отмена</button>
      <button class="delete" @click="confirmDelete">Удалить</button>
    </template>
  </Modal>
</template>
Пример 2: Карточка товара

<!-- ProductCard.vue -->
<template>
  <div class="product-card">
    <div class="product-image">
      <slot name="image">
        <img :src="product.image" :alt="product.name">
      </slot>
    </div>
    
    <div class="product-info">
      <slot name="title">
        <h3>{{ product.name }}</h3>
      </slot>
      
      <slot name="price">
        <p class="price">{{ formatPrice(product.price) }}</p>
      </slot>
      
      <slot name="description">
        <p class="description">{{ product.description }}</p>
      </slot>
      
      <slot name="actions">
        <button @click="addToCart">В корзину</button>
      </slot>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  product: Object
})

const formatPrice = (price) => {
  return new Intl.NumberFormat('ru-RU', {
    style: 'currency',
    currency: 'RUB'
  }).format(price)
}

const addToCart = () => {
  console.log('Добавлено в корзину:', props.product)
}
</script>

<!-- Расширенное использование -->
<template>
  <ProductCard :product="product">
    <template #title>
      <div class="custom-title">
        <h3>{{ product.name }}</h3>
        <span class="badge" v-if="product.isNew">Новинка!</span>
      </div>
    </template>
    
    <template #price>
      <div class="price-block">
        <span class="old-price">{{ formatPrice(product.oldPrice) }}</span>
        <span class="current-price">{{ formatPrice(product.price) }}</span>
        <span class="discount">-{{ discount }}%</span>
      </div>
    </template>
    
    <template #actions="{ product }">
      <div class="actions">
        <button @click="buyNow(product">Купить сейчас</button>
        <button @click="addToWishlist(product)">❤️ В избранное</button>
      </div>
    </template>
  </ProductCard>
</template>
Пример 3: Форма с валидацией

<!-- FormWrapper.vue -->
<template>
  <form @submit.prevent="handleSubmit" class="form-wrapper">
    <slot name="before"></slot>
    
    <div class="form-fields">
      <slot></slot>
    </div>
    
    <div class="form-actions">
      <slot name="actions">
        <button type="submit" :disabled="!isValid">
          {{ submitText }}
        </button>
        <button type="button" @click="reset">Сбросить</button>
      </slot>
    </div>
    
    <slot name="after"></slot>
  </form>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  isValid: Boolean,
  submitText: {
    type: String,
    default: 'Отправить'
  }
})

const emit = defineEmits(['submit', 'reset'])

const handleSubmit = () => {
  if (props.isValid) {
    emit('submit')
  }
}

const reset = () => {
  emit('reset')
}
</script>

<!-- Использование с FormKit или VeeValidate -->
<template>
  <FormWrapper :is-valid="meta.valid" @submit="onSubmit" @reset="resetForm">
    <div class="field">
      <label>Имя</label>
      <Field name="name" rules="required" v-slot="{ field, errors }">
        <input v-bind="field" type="text" />
        <span class="error">{{ errors[0] }}</span>
      </Field>
    </div>
    
    <div class="field">
      <label>Email</label>
      <Field name="email" rules="required|email" v-slot="{ field, errors }">
        <input v-bind="field" type="email" />
        <span class="error">{{ errors[0] }}</span>
      </Field>
    </div>
    
    <template #actions>
      <button type="submit" class="custom-submit">
        Зарегистрироваться
      </button>
      <button type="button" class="custom-cancel" @click="cancel">
        Отмена
      </button>
    </template>
  </FormWrapper>
</template>
 Продвинутые техники
1. Рендеринг слотов через $slots

<script setup>
import { useSlots } from 'vue'

const slots = useSlots()

// Проверка наличия слота
const hasHeader = computed(() => !!slots.header)

// Условный рендеринг
</script>

<template>
  <div>
    <div v-if="hasHeader" class="header">
      <slot name="header" />
    </div>
    <div v-else class="default-header">
      Заголовок по умолчанию
    </div>
  </div>
</template>
2. Передача слотов через props

<!-- Child.vue -->
<template>
  <div>
    <slot name="content" :data="localData"></slot>
  </div>
</template>

<!-- Parent.vue -->
<template>
  <Child>
    <template #content="{ data }">
      <GrandChild>
        <template #default>
          {{ data }}
        </template>
      </GrandChild>
    </template>
  </Child>
</template>
3. Утилита $slot для компонентов-оберток

<!-- Card.vue -->
<template>
  <div class="card" :class="variant">
    <div v-if="$slots.header" class="card-header">
      <slot name="header" />
    </div>
    
    <div class="card-content">
      <slot />
    </div>
    
    <div v-if="$slots.footer" class="card-footer">
      <slot name="footer" />
    </div>
  </div>
</template>

<script setup>
defineProps({
  variant: {
    type: String,
    default: 'default',
    validator: (value) => ['default', 'primary', 'danger'].includes(value)
  }
})
</script>
