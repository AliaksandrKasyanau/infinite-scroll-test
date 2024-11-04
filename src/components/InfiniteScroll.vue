<script lang="ts" setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

interface User {
  name: {
    first: string
    last: string
  }
  email: string
  picture: {
    medium: string
  }
}

interface ApiResponse {
  results: User[]
}

const users = ref<User[]>([])
const loading = ref<boolean>(false)
const sentry = ref<Element | null>(null)

let page: number = 1
const limit: number = 10

const loadMoreUsers = async (): Promise<void> => {
  loading.value = true
  try {
    const response = await fetch(`https://randomuser.me/api/?page=${page}&results=${limit}`)
    const data: ApiResponse = await response.json()
    users.value.push(...data.results)
    page += 1
  } catch (error) {
    console.error('Ошибка при загрузке пользователей:', error)
  } finally {
    loading.value = false
  }
}

let observer: IntersectionObserver | undefined

const handleIntersect = (entries: IntersectionObserverEntry[]): void => {
  if (entries[0].isIntersecting && !loading.value) {
    loadMoreUsers()
  }
}

onMounted(() => {
  observer = new IntersectionObserver(handleIntersect)
  if (sentry.value) observer.observe(sentry.value)
})

onBeforeUnmount(() => {
  if (observer) {
    observer.disconnect()
  }
})
</script>

<template>
  <div class="cards">
    <article v-for="user in users" :key="user.email" class="card">
      <img :src="user.picture.medium" alt="User picture" class="card__image" />
      <div class="card__info">
        <h3 class="card__name">{{ user.name.first }} {{ user.name.last }}</h3>
        <p class="card__email">{{ user.email }}</p>
      </div>
    </article>
    <div v-if="loading" class="loading">Fetching users...</div>
    <div ref="sentry" class="sentry"></div>
  </div>
</template>

<style scoped lang="scss">
.cards {
  margin: 0 auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  pointer-events: none;
}

.card {
  max-width: 20rem;
  display: flex;
  flex-direction: row;
  gap: 0.5rem;
  cursor: pointer;
  color: #ffffffda;
  padding: 0;
  background-color: #202022;
  pointer-events: auto;
  transform: scale(1);
  opacity: 1;
  transition: all 150ms ease-in-out;
  border: 3px solid #484848;
  border-radius: 0.75rem;
  overflow: hidden;
  text-overflow: ellipsis;

  .card__image {
    width: 5rem;
    border-top-left-radius: inherit;
    border-bottom-left-radius: inherit;
    border-right: 3px solid #484848;
    object-fit: cover;
  }

  .card__info {
    display: flex;
    flex-direction: column;
    align-self: center;
    gap: 0.25rem;
    padding: 0.25rem;

    .card__name {
      margin: 0;
      display: flex;
      text-overflow: ellipsis;
      overflow: hidden;
    }

    .card__email {
      font-size: 12px;
      text-overflow: ellipsis;
      overflow: hidden;
    }
  }

  &:nth-child(even) {
    transform: translateY(8px);
  }

  &:nth-child(n) {
    transform: rotate(-5deg);
  }
}

.cards:hover>.card:hover {
  z-index: 10;
  transform: scale(1.15);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.cards:hover>.card:not(:hover) {
  opacity: 0.5;
}

.loading {
  text-align: center;
  font-size: 1.2em;
  padding: 15px;
}

.sentry {
  height: 1px;
}
</style>
