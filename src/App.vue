<script setup>
import { onMounted, reactive, ref, watch, watchEffect } from "vue";
import { EditOutlined, DeleteOutlined } from "@ant-design/icons-vue";
import { toast } from "vue3-toastify";
import axios from "axios";
import orderBy from "lodash/orderBy";

import "vue3-toastify/dist/index.css";

const state = reactive({
  users: [],
  isVisibleModal: false,
  email: "",
  firstName: "",
  lastName: "",
  selectedUserId: null,
});

const filterValue = ref("");

const sortValue = ref("id");

const onEditClick = (userId) => {
  state.selectedUserId = userId;
  toggleModal();
};

const toggleModal = () => {
  state.isVisibleModal = !state.isVisibleModal;
  state.email = "";
  state.firstName = "";
  state.lastName = "";
};

const onEditCard = async () => {
  try {
    await axios.patch(`https://reqres.in/api/users/${state.selectedUserId}`, {
      email: state.email,
      first_name: state.firstName,
      last_name: state.lastName,
    });
    toast.success("Данные пользователя успешно редактированы", {
      position: "top-left",
      autoClose: 2000,
    });
    toggleModal();
  } catch (error) {
    toast.error("Ошибка при редактировании пользователя", {
      position: "top-left",
      autoClose: 2000,
    });
  }
};

const onDeleteCard = async (user) => {
  try {
    if (window.confirm("Вы действительно хотите удалить карточку?")) {
      await axios.delete(`https://reqres.in/api/users/${user.id}`);
      toast.success(
        `Пользователь ${user.first_name} ${user.last_name} успешно удален`,
        {
          position: "top-left",
          autoClose: 2000,
        }
      );
    }
  } catch (error) {
    toast.error("Ошибка при удалении пользователя", {
      position: "top-left",
      autoClose: 2000,
    });
  }
};

onMounted(async () => {
  try {
    const { data } = await axios.get("https://reqres.in/api/users");

    state.users = data.data;
    toast.success("Данные пользователей успешно получены", {
      position: "top-left",
      autoClose: 2000,
    });
  } catch (error) {
    toast.error("Ошибка при получении списка пользователей", {
      position: "top-left",
    });
  }
});

watchEffect(() => {
  if (sortValue.value === "id") {
    state.users = orderBy(state.users, "id", "asc");
  } else if (sortValue.value === "firstName") {
    state.users = orderBy(state.users, "first_name", "asc");
  } else if (sortValue.value === "lastName") {
    state.users = orderBy(state.users, "last_name", "asc");
  } else if (sortValue.value === "email") {
    state.users = orderBy(state.users, "email", "asc");
  }
});

watch(filterValue, async (newValue) => {
  const { data } = await axios.get("https://reqres.in/api/users");

  state.users = data.data.filter(
    (item) =>
      item.email.toLowerCase().indexOf(newValue) >= 0 ||
      item.first_name.toLowerCase().indexOf(newValue) >= 0 ||
      item.last_name.toLowerCase().indexOf(newValue) >= 0
  );
});
</script>

<template>
  <div class="container">
    <div class="toolbar">
      <div class="toolbar__input">
        <a-input
          type="text"
          placeholder="Поиск"
          style="width: 300px"
          v-model:value="filterValue"
        />
      </div>
      <div class="toolbar__sort">
        <a-radio-group style="margin-bottom: 16px" v-model:value="sortValue">
          <a-radio-button value="id">Id</a-radio-button>
          <a-radio-button value="email">Email</a-radio-button>
          <a-radio-button value="firstName">Firstname</a-radio-button>
          <a-radio-button value="lastName">Lastname</a-radio-button>
        </a-radio-group>
      </div>
    </div>

    <div class="cards">
      <a-card
        v-for="user in state.users"
        :key="user.id"
        style="width: 300px; margin-bottom: 30px"
      >
        <template #actions>
          <edit-outlined key="edit" @click="() => onEditClick(user.id)" />
          <delete-outlined key="delete" @click="() => onDeleteCard(user)" />
        </template>
        <p>id : {{ user.id }}</p>
        <p>email : {{ user.email }}</p>
        <p>firstname : {{ user.first_name }}</p>
        <p>lastname : {{ user.last_name }}</p>
      </a-card>
    </div>

    <a-modal v-model:open="state.isVisibleModal" @ok="onEditCard">
      <a-input
        type="text"
        placeholder="Email"
        v-model:value="state.email"
        style="width: 300px; margin-bottom: 15px; padding: 10px"
      />
      <a-input
        type="text"
        placeholder="Firstname"
        v-model:value="state.firstName"
        style="width: 300px; margin-bottom: 15px; padding: 10px"
      />
      <a-input
        type="text"
        placeholder="Lastname"
        v-model:value="state.lastName"
        style="width: 300px; margin-bottom: 15px; padding: 10px"
      />
    </a-modal>
  </div>
</template>

<style scoped>
.container {
  max-width: 1140px;
  margin: 0 auto;
  margin-top: 15px;
  padding: 15px;
}

.cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-top: 30px;
}

.filter-input {
  padding: 5px 10px;
}

.toolbar {
  display: flex;
  flex-direction: column;
}

.toolbar__sort {
  margin-top: 30px;
}

@media screen and (max-width: 575px) {
  .toolbar {
    align-items: center;
  }
  .cards {
    justify-content: center;
  }
}
</style>
