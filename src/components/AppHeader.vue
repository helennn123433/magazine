<template>
  <header>
    <div class="info">
      <div class="profile">
        <img src="@/assets/Union.svg" alt="profile" />
        <router-link v-if="!user" to="/signin">
          <n-button style="--n-border-hover: 1px solid #fff">Войти</n-button>
        </router-link>
        <div v-else>
          <span>{{ user.sub }}</span>
          <n-button @click="logout" style="--n-border-hover: 1px solid #fff">
            Выйти
            </n-button>
        </div>
      </div>
      <div class="cart">
        <img src="@/assets/cart.svg" alt="cart" />
        <router-link to="/cart">
          <n-button style="--n-border-hover: 1px solid #fff">Корзина</n-button>
        </router-link>
        <span>{{ cartItemCount }}</span>
      </div>
      <div class="main">
        <router-link to="/">
          <n-button
            style="
              --n-border-hover: 1px solid #fff;
              --n-text-color-focus: #f0f0f0;
              --n-border-focus: 1px solid #f0f0f0;">
            Главная</n-button>
        </router-link>
      </div>
    </div>
    <div class="menu">
      <n-button
        style="--n-border-hover: 1px solid #fff"
        @click="$emit('toggle-drawer')">
        Каталог
      </n-button>
    </div>
  </header>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue';
import { useCartStore } from '@/store/cartStore';
import { useUserStore } from '@/store/userStore';
import { useRouter } from 'vue-router';
import { NButton } from 'naive-ui';

const router = useRouter();
const cartStore = useCartStore();
const userStore = useUserStore();
const cartItemCount = ref(0);

const user = computed(() => userStore.user.value);

const logout = () => {
  userStore.logout();
  router.push("/signin");
};

watch(
  () => cartStore.cartItems,
  (newItems) => {
    cartItemCount.value = newItems.reduce(
      (total, item) => total + item.count, 0);
  },
  { immediate: true, deep: true }
);

onMounted(async () => {
  await userStore.fetchUser();
  if (user.value) {
    const token = localStorage.getItem("token");
    await cartStore.fetchCart(user.value.id, token);
  }
});
</script>

<style scoped>
header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 40px;
  background-color: #465a86;
}
.info {
  display: flex;
  gap: 30px;
}
.profile,
.cart,
.menu {
  display: flex;
  align-items: center;
  gap: 10px;
}
.profile span {
  color: #fff;
  margin-right: 10px;
}
.cart span {
  width: 40px;
  color: #fff;
}
.n-button {
  color: #fff;
}
.n-button:hover {
  color: #f0f0f0;
}
</style>
