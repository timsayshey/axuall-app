<template>
  <div class="user-directory">
    <div class="container">
      <div class="d-flex justify-content-center">
        <h1>User Directory</h1>
      </div>
      <div class="d-flex justify-content-center pb-5">
        <a @click="downloadCSV()" class="btn btn-secondary btn-sm"
          >Export CSV</a
        >
      </div>
      <div class="row mb-2 users">
        <div
          class="col-md-6 user-item"
          v-for="user in users"
          :key="user.id.value"
        >
          <div
            class="row g-0 border rounded overflow-hidden flex-md-row mb-4 shadow-sm h-md-250 position-relative"
          >
            <div class="col-sm-12 col-lg-4 text-center pt-4">
              <img class="rounded-circle" :src="user.picture.large" />
            </div>
            <div class="col p-4 d-flex flex-column position-static">
              <h3 class="mb-0">{{ user.name.first }} {{ user.name.last }}</h3>
              <div class="mb-1 text-muted">
                <a :href="`mailto:${user.email}`">{{ user.email }}</a>
              </div>
              <div class="row">
                <div class="col-12">
                  <i class="fa-solid fa-phone me-1"></i>
                  <a :href="`tel:${user.phone}`">{{ user.phone }}</a>
                </div>
                <div class="col-12">
                  <i class="fa-regular fa-clock"></i> {{ user.dob.age }} years
                  old
                </div>
                <div class="col-12">
                  <i class="fa-solid fa-user gender-icon"></i>
                  <p class="capitalize-first">{{ user.gender }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="d-flex justify-content-center">
        <nav aria-label="pagination">
          <ul class="pagination">
            <li class="page-item" :class="{ disabled: currentPage <= 1 }">
              <span class="page-link" @click="currentPage -= 1">Previous</span>
            </li>
            <li
              v-for="page in pageList"
              :key="page"
              class="page-item"
              :class="{ active: page === currentPage }"
            >
              <a class="page-link" @click="currentPage = page">{{ page }}</a>
            </li>
            <li
              class="page-item"
              :class="{ disabled: currentPage >= totalPages }"
            >
              <a class="page-link" @click="currentPage += 1">Next</a>
            </li>
          </ul>
        </nav>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { ref, watch } from "vue";
import { onMounted } from "@vue/runtime-core";
export default {
  name: "UserDirectory",
  props: {},
  setup() {
    const totalPages = ref(6);
    const currentPage = ref(1);
    const users = ref();
    const getUsers = async () => {
      const res = await axios.get("https://randomuser.me/api/", {
        params: {
          results: 10,
          page: currentPage.value,
          seed: "abc",
        },
      });
      return res.data.results;
    };
    const downloadCSV = () => {
      const headers = "first_name,last_name,email,phone,age,gender";
      let csvContent = `data:text/csv;charset=utf-8,${headers}\n`;
      users.value.forEach((user) => {
        csvContent += `${user.name.first},${user.name.last},${user.email},${user.phone},${user.dob.age},${user.gender}\n`;
      });
      var encodedUri = encodeURI(csvContent);
      var link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", "user_export.csv");
      document.body.appendChild(link); // Required for FF
      link.click();
    };
    onMounted(async () => {
      users.value = await getUsers();
    });
    watch(currentPage, async () => {
      users.value = await getUsers();
    });
    return {
      downloadCSV,
      totalPages,
      currentPage,
      pageList: Array.from({ length: totalPages.value }, (_, i) => i + 1), // generate array of page numbers
      users,
    };
  },
};
</script>

<style lang="scss" scoped>
.user-directory {
  padding: 40px 10px;
  img {
    min-height: 128px;
  }
  .users a {
    color: #000;
    text-decoration: underline;
    &:hover {
      text-decoration: none;
    }
  }
  .gender-icon {
    float: left;
    margin-right: 5px;
  }
  .capitalize-first {
    text-transform: lowercase;
  }
  .capitalize-first::first-letter {
    text-transform: uppercase;
  }
  .page-link {
    cursor: pointer;
    user-select: none;
  }
  .user-item {
    min-height: 199px;
  }
}
</style>
