<template>
  <div class="container">
    <header class="header">
      <h3>Nhân viên</h3>
      <button class="btn btn-primary" @click="addUser">
        Thêm mới nhân viên
      </button>
    </header>

    <div class="search-bar">
      <input
        type="text"
        class="form-control"
        placeholder="Tìm kiếm theo email"
        v-model="searchQuery"
      />
    </div>

    <table class="table table-bordered table-hover table-striped">
      <thead>
        <tr>
          <th>STT</th>
          <th>Họ và tên</th>
          <th>Ngày sinh</th>
          <th>Email</th>
          <th>Địa chỉ</th>
          <th>Trạng thái</th>
          <th colspan="2">Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in filteredUsers" :key="item.id">
          <td>{{ index + 1 }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.birthday }}</td>
          <td>{{ item.email }}</td>
          <td>{{ item.address }}</td>
          <td>
            <span
              :class="{
                'status-active': item.status,
                'status-inactive': !item.status,
              }"
            >
              {{ item.status ? "đang hoạt động" : "không hoạt động" }}
            </span>
          </td>
          <td>
            <button class="btn btn-warning" @click="handleUpdateUser(item)">
              Sửa
            </button>
          </td>
          <td>
            <button class="btn btn-danger" @click="handleDeleteModal(item)">
              Xóa
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <footer class="pagination-footer">
      <select v-model="recordsPerPage" class="form-select">
        <option value="10">Hiển thị 10 bản ghi</option>
        <option value="20">Hiển thị 20 bản ghi</option>
        <option value="50">Hiển thị 50 bản ghi</option>
        <option value="100">Hiển thị 100 bản ghi</option>
      </select>
      <ul class="pagination">
        <li class="page-item" :class="{ disabled: currentPage === 1 }">
          <a class="page-link" @click.prevent="previousPage">Trước</a>
        </li>
        <li class="page-item" v-for="page in totalPages" :key="page">
          <a class="page-link" @click.prevent="changePage(page)">{{ page }}</a>
        </li>
        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
          <a class="page-link" @click.prevent="nextPage">Sau</a>
        </li>
      </ul>
    </footer>

    <div v-if="activateForm" class="modal-overlay">
      <div class="modal-content">
        <h4>
          {{
            checkStatusForm === "add"
              ? "Thêm mới nhân viên"
              : "Chỉnh sửa nhân viên"
          }}
        </h4>
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label for="userName">Họ và tên</label>
            <input
              v-model="initialUser.name"
              id="userName"
              type="text"
              class="form-control"
            />
            <span class="error-message" v-if="initialError.name">{{
              initialError.name
            }}</span>
          </div>

          <div class="form-group">
            <label for="dateOfBirth">Ngày sinh</label>
            <input
              v-model="initialUser.birthday"
              id="dateOfBirth"
              type="date"
              class="form-control"
            />
            <span class="error-message" v-if="initialError.birthday">{{
              initialError.birthday
            }}</span>
          </div>

          <div class="form-group">
            <label for="email">Email</label>
            <input
              v-model="initialUser.email"
              id="email"
              type="text"
              class="form-control"
            />
            <span class="error-message" v-if="initialError.email">{{
              initialError.email
            }}</span>
          </div>

          <div class="form-group">
            <label for="address">Địa chỉ</label>
            <textarea
              v-model="initialUser.address"
              class="form-control"
              id="address"
            ></textarea>
          </div>

          <div class="form-group">
            <button type="submit" class="btn btn-primary">
              {{ checkStatusForm === "add" ? "Thêm mới" : "Cập nhật" }}
            </button>
            <button
              type="button"
              class="btn btn-secondary"
              @click="handleCloseForm"
            >
              Hủy
            </button>
          </div>
        </form>
      </div>
    </div>

    <div v-if="deleteModal" class="modal-overlay">
      <div class="modal-content">
        <h4>Cảnh báo</h4>
        <p>Bạn có chắc chắn muốn xóa tài khoản này?</p>
        <div class="modal-actions">
          <button class="btn btn-light" @click="handleCloseDelete">Hủy</button>
          <button class="btn btn-danger" @click="handleDelete">Xác nhận</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed } from "vue";

const activateForm = ref(false);
const checkStatusForm = ref("add");
const users = reactive(JSON.parse(localStorage.getItem("users")) || []);
const initialUser = reactive({
  id: 0,
  name: "",
  birthday: "",
  email: "",
  address: "",
  status: true,
});
const initialError = reactive({ name: "", birthday: "", email: "" });
const deleteModal = ref(false);
const deleteUser = ref();
const searchQuery = ref("");
const recordsPerPage = ref(10);
const currentPage = ref(1);

const resetError = () => {
  initialError.name = "";
  initialError.birthday = "";
  initialError.email = "";
};
const resetUser = () => {
  initialUser.id = 0;
  initialUser.name = "";
  initialUser.birthday = "";
  initialUser.email = "";
  initialUser.address = "";
  initialUser.status = true;
};
const validateEmail = (email) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
const validateDate = (selectedDate) => {
  const currentDate = new Date();
  const chosenDate = new Date(selectedDate);
  return chosenDate <= currentDate;
};

const validateUser = () => {
  resetError();
  let isValid = true;
  if (initialUser.name === "") {
    initialError.name = "Tên không được để trống";
    isValid = false;
  }
  if (initialUser.email === "") {
    initialError.email = "Email không được để trống";
    isValid = false;
  } else if (!validateEmail(initialUser.email)) {
    initialError.email = "Email không hợp lệ";
    isValid = false;
  }
  if (initialUser.birthday === "") {
    initialError.birthday = "Ngày sinh không được để trống";
    isValid = false;
  } else if (!validateDate(initialUser.birthday)) {
    initialError.birthday = "Ngày sinh không được vượt qua ngày hiện tại";
    isValid = false;
  }
  return isValid;
};

const handleSubmit = () => {
  if (validateUser()) {
    if (checkStatusForm.value === "add") {
      initialUser.id = Math.ceil(Math.random() * 100001);
      users.push({ ...initialUser });
    } else if (checkStatusForm.value === "edit") {
      const index = users.findIndex((user) => user.id === initialUser.id);
      users.splice(index, 1, { ...initialUser });
    }
    localStorage.setItem("users", JSON.stringify(users));
    activateForm.value = false;
    resetUser();
  }
};

const blockUser = (item) => {
  item.status = !item.status;
  localStorage.setItem("users", JSON.stringify(users));
};

const handleUpdateUser = (item) => {
  Object.assign(initialUser, item);
  activateForm.value = true;
  checkStatusForm.value = "edit";
};

const handleDeleteModal = (item) => {
  deleteModal.value = true;
  deleteUser.value = item;
};
const handleDelete = () => {
  const index = users.findIndex((user) => user.id === deleteUser.value.id);
  users.splice(index, 1);
  localStorage.setItem("users", JSON.stringify(users));
  deleteModal.value = false;
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.search-bar {
  margin-bottom: 20px;
}

.table {
  width: 100%;
}

.status-active {
  color: green;
}

.status-inactive {
  color: red;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  width: 500px;
  max-width: 90%;
}

.error-message {
  color: red;
  font-size: 12px;
}

.pagination-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 20px;
}
</style>
