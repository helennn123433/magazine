<template>
  <div v-if="isVisible" class="catalog">
    <div class="cart">
      <h2>Каталог товаров</h2>
      <svg
        @click="$emit('close-drawer')" width="16" height="14" viewBox="0 0 16 14" fill="#fff" xmlns="http://www.w3.org/2000/svg">
        <path d="M1 7H14.7143" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        <path d="M8.71436 1L14.7144 7L8.71436 13" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </div>
    <n-button
      v-if="!editMode && isAdmin"
      type="primary"
      size="small"
      @click="enableEditMode"
    >
      Включить режим редактирования
    </n-button>

    <Draggable
      v-model="categories"
      :group="{ name: 'categories' }"
      @end="onDragEnd"
      item-key="name"
      :disabled="!editMode"
    >
      <template #item="{ element }">
        <li>
          <router-link
            :to="{ name: 'Category', params: { categoryName: element.name } }">
            <strong>{{ element.name }}</strong>
          </router-link>

          <Draggable
            v-model="element.subcategories"
            :group="{ name: 'subcategories' }"
            item-key="name"
            :disabled="!editMode"
          >
            <template #item="{ element: subcategory }">
              <ul>
                <li>
                  <router-link
                    :to="{
                      name: 'Category',
                      params: {
                        categoryName: element.name,
                        subcategoryName: subcategory.name,
                      },
                    }">
                    <strong>{{ subcategory.name }}</strong>
                  </router-link>

                  <Draggable
                    v-model="subcategory.subsubcategories"
                    :group="{ name: 'subsubcategories' }"
                    item-key="name"
                    :disabled="!editMode"
                  >
                    <template #item="{ element: subsubcategory }">
                      <ul>
                        <li>
                          <router-link
                            :to="{
                              name: 'Category',
                              params: {
                                categoryName: element.name,
                                subcategoryName: subcategory.name,
                                subsubcategoryName: subsubcategory.name,
                              },
                            }">
                            <strong>{{ subsubcategory.name }}</strong>
                          </router-link>

                          <Draggable
                            v-model="subsubcategory.products"
                            :group="{ name: 'products' }"
                            item-key="id"
                            :disabled="!editMode"
                          >
                            <template #item="{ element: product }">
                              <ul>
                                <li>
                                  <router-link
                                    :to="{
                                      name: 'ProductView',
                                      params: { productId: product.id },
                                    }">
                                    {{ product.title }}
                                  </router-link>
                                </li>
                              </ul>
                            </template>
                          </Draggable>
                        </li>
                      </ul>
                    </template>
                  </Draggable>

                  <Draggable
                    v-model="subcategory.products"
                    :group="{ name: 'products' }"
                    item-key="id"
                    :disabled="!editMode"
                  >
                    <template #item="{ element: product }">
                      <ul>
                        <li>
                          <router-link
                            :to="{
                              name: 'ProductView',
                              params: { productId: product.id },
                            }">
                            {{ product.title }}
                          </router-link>
                        </li>
                      </ul>
                    </template>
                  </Draggable>
                </li>
              </ul>
            </template>
          </Draggable>

          <Draggable
            v-model="element.productsWithoutSubcategory"
            :group="{ name: 'products' }"
            item-key="id"
            :disabled="!editMode"
          >
            <template #item="{ element: product }">
              <ul>
                <li>
                  <router-link
                    :to="{
                      name: 'ProductView',
                      params: { productId: product.id },
                    }">
                    {{ product.title }}
                  </router-link>
                </li>
              </ul>
            </template>
          </Draggable>
        </li>
      </template>
    </Draggable>

    <n-button v-if="editMode" type="warning" @click="saveOrder">
      Сохранить изменения
    </n-button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, defineProps } from "vue";
import axios from "axios";
import Draggable from "vuedraggable";
import { NButton } from "naive-ui";
import { useUserStore } from "@/store/userStore";
import { useOrganizeProducts } from "@/composables/useOrganizeProducts";

const userStore = useUserStore();
const categories = ref([]);
const editMode = ref(false);
const { organizeProductsByCategories } = useOrganizeProducts();

defineProps({
  isVisible: Boolean,
});

const enableEditMode = () => {
  editMode.value = true;
};

const fetchData = async () => {
  try {
    const response = await axios.get("http://localhost:8080/product/getAll");
    const products = response.data;
    if (Array.isArray(products)) {
      categories.value = organizeProductsByCategories(products);
    } else {
      console.error("Неправильный формат данных:", products);
    }
  } catch (error) {
    console.error("Ошибка при получении данных:", error);
  }
};

const collectChanges = () => {
  let changes = {};

  const processCategory = (category, categoryIndex) => {
    category.productsWithoutSubcategory.forEach((product, productIndex) => {
      if (!changes[product.id]) {
        changes[product.id] = [];
      }
      changes[product.id].push({
        name: category.name,
        subcategory: null,
        subsubcategory: null,
        order: categoryIndex + 1,
        subcategoryOrder: null,
        subsubcategoryOrder: null,
        productOrder: productIndex + 1,
      });
    });

    category.subcategories.forEach((subcategory, subcategoryIndex) => {
      subcategory.products.forEach((product, productIndex) => {
        if (!changes[product.id]) {
          changes[product.id] = [];
        }
        changes[product.id].push({
          name: category.name,
          subcategory: subcategory.name,
          subsubcategory: null,
          order: categoryIndex + 1,
          subcategoryOrder: subcategoryIndex + 1,
          subsubcategoryOrder: null,
          productOrder: productIndex + 1,
        });
      });

      subcategory.subsubcategories.forEach(
        (subsubcategory, subsubcategoryIndex) => {
          subsubcategory.products.forEach((product, productIndex) => {
            if (!changes[product.id]) {
              changes[product.id] = [];
            }
            changes[product.id].push({
              name: category.name,
              subcategory: subcategory.name,
              subsubcategory: subsubcategory.name,
              order: categoryIndex + 1,
              subcategoryOrder: subcategoryIndex + 1,
              subsubcategoryOrder: subsubcategoryIndex + 1,
              productOrder: productIndex + 1,
            });
          });
        }
      );
    });
  };

  categories.value.forEach((category, index) => {
    processCategory(category, index);
  });

  return changes;
};

const saveOrder = async () => {
  try {
    const changes = collectChanges();
    const requests = Object.keys(changes).map((productId) =>
      axios.put("http://localhost:8080/product/categories/reorder", {
        [productId]: changes[productId],
      })
    );
    await Promise.all(requests);
    console.log("Изменения успешно сохранены.");
  } catch (error) {
    console.error("Ошибка при сохранении порядка:", error);
  }
};

const role = computed(() => userStore.role.value);
const isAdmin = computed(() => role.value === "ROLE_ADMIN");

onMounted(() => {
  userStore.fetchUser();
  fetchData();
});
</script>
<style scoped>
.catalog {
  display: flex;
  flex-direction: column;
  position: fixed;
  height: 100%;
  width: 24rem;
  background-color: #465a86;
  padding: 1.75rem 2.5rem;
  top: 0;
  right: 0;
  z-index: 10;
  border-left: 3px solid #f0f0f0;
}
.catalog a {
  color: #fff;
}

h2,
span {
  font-family: Arial, Helvetica, sans-serif;
  color: #fff;
}

.cart {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

svg {
  cursor: pointer;
}

ul {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

a {
  text-decoration: none;
  color: black;
  display: block;
}

a:focus {
  outline: none;
}
a:hover {
  color: #007bff;
}

li {
  margin-bottom: 0.5rem;
  padding: 0.5rem;
  border-radius: 4px;
}

ul ul {
  padding-left: 1rem;
}

.catalog:hover ul,
.catalog:hover a {
  background-color: transparent;
}
</style>
