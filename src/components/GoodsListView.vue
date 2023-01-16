<script setup>
import axios from "axios";
import { Modal } from "bootstrap";
import { onMounted, ref } from "vue";
const apiUrl = "https://vue3-course-api.hexschool.io";
const token = localStorage.getItem("token");
const goodsData = ref([]);
const showData = ref([]);
const modalTarget = ref({});
const modalImageUrl = ref("");
const imageOnChange = ref("");
const productModal = ref({});
const delProductModal = ref({});

onMounted(() => {
  productModal.value = new Modal(document.querySelector("#productModal"));
  delProductModal.value = new Modal(document.querySelector("#delProductModal"));
});

function getGoodsData() {
  axios
    .get(`${apiUrl}/v2/api/thisisastore/admin/products/all`, {
      headers: { Authorization: token },
    })
    .then((res) => {
      goodsData.value = [];
      let originData = Object.entries(res.data.products);
      originData.forEach((item) => {
        goodsData.value.push(item[1]);
      });
      goodsDataProcess();
    })
    .catch((err) => {
      console.log(err);
    });
}

function sendProductData() {
  if (modalTarget.value.id) {
    axios
      .put(
        `${apiUrl}/v2/api/thisisastore/admin/product/${modalTarget.value.id}`,
        { data: modalTarget.value },
        {
          headers: { Authorization: token },
        }
      )
      .then((res) => {
        console.log(res);
        getGoodsData();
        productModal.value.hide();
        alert(`${res.data.message}`);
      })
      .catch((err) => {
        console.log(err);
      });
  } else {
    axios
      .post(
        `${apiUrl}/v2/api/thisisastore/admin/product`,
        { data: modalTarget.value },
        {
          headers: { Authorization: token },
        }
      )
      .then((res) => {
        console.log(res);
        getGoodsData();
        productModal.value.hide();
        alert(`${res.data.message}`);
      })
      .catch((err) => {
        console.log(err);
      });
  }
}

function deleteProduct() {
  axios
    .delete(
      `${apiUrl}/v2/api/thisisastore/admin/product/${modalTarget.value.id}`,
      {
        headers: { Authorization: token },
      }
    )
    .then((res) => {
      console.log(res);
      getGoodsData();
      delProductModal.value.hide();
      alert(`${res.data.message}`);
    })
    .catch((err) => {
      console.log(err);
    });
}

function goodsDataProcess() {
  showData.value = "";
  showData.value = goodsData.value;
}

function modalDataRender(targetId) {
  modalTarget.value = "";
  modalTarget.value = showData.value.find((item) => item.id === targetId);
  modalImageUrl.value = modalTarget.value.imageUrl;
}

function imageAdd() {
  if (!modalTarget.value.imagesUrl) modalTarget.value.imagesUrl = [];
  let imgIsExisted = modalTarget.value.imagesUrl.find(
    (item) => item === modalImageUrl.value
  );

  if (!modalTarget.value.imageUrl) {
    modalTarget.value.imageUrl = modalImageUrl.value;
  } else if (
    modalImageUrl.value != modalTarget.value.imageUrl &&
    imgIsExisted === undefined
  ) {
    modalTarget.value.imagesUrl.push(modalImageUrl.value);
  } else {
    return;
  }
}

function imageEdit() {
  if (imageOnChange.value === "main") {
    modalTarget.value.imageUrl = modalImageUrl.value;
  } else {
    modalTarget.value.imagesUrl[imageOnChange.value] = modalImageUrl.value;
  }
}

getGoodsData();
</script>

<template>
  <div class="container">
    <div class="text-end mt-4">
      <button
        data-bs-toggle="modal"
        data-bs-target="#productModal"
        class="btn btn-primary"
        @click.prevent="
          modalTarget = {};
          modalImageUrl = '';
        "
      >
        建立新的產品
      </button>
    </div>
    <table class="table mt-4">
      <thead>
        <tr>
          <th width="120">分類</th>
          <th>產品名稱</th>
          <th width="120">原價</th>
          <th width="120">售價</th>
          <th width="100">是否啟用</th>
          <th width="120">編輯</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in showData" :key="item.id">
          <td>{{ item.category }}</td>
          <td>{{ item.title }}</td>
          <td class="text-end">{{ item.origin_price }}</td>
          <td class="text-end">{{ item.price }}</td>
          <td>
            <span v-if="item.is_enabled" class="text-success">啟用</span>
            <span v-else class="text-danger">未啟用</span>
          </td>
          <td>
            <div class="btn-group">
              <button
                type="button"
                class="btn btn-outline-primary btn-sm"
                data-bs-toggle="modal"
                data-bs-target="#productModal"
                @click.prevent="modalDataRender(item.id)"
              >
                編輯
              </button>
              <button
                type="button"
                class="btn btn-outline-danger btn-sm"
                data-bs-toggle="modal"
                data-bs-target="#delProductModal"
                @click.prevent="modalDataRender(item.id)"
              >
                刪除
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <!-- Modal -->
  <div
    id="productModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="productModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog modal-xl">
      <div class="modal-content border-0">
        <div class="modal-header bg-dark text-white">
          <h5 id="productModalLabel" class="modal-title">
            <span>{{ modalTarget.value != {} ? "編輯產品" : "新增產品" }}</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          <div class="row">
            <div class="col-sm-4">
              <div class="mb-2">
                <div class="mb-3">
                  <label for="imageUrl" class="form-label">{{
                    modalTarget.imageUrl ? "圖片網址" : "輸入圖片網址"
                  }}</label>
                  <input
                    v-model="modalImageUrl"
                    type="text"
                    class="form-control"
                    placeholder="請輸入圖片連結"
                  />
                </div>
                <img class="img-fluid" :src="modalImageUrl" alt="" />
              </div>
              <div v-if="modalTarget.imageUrl" class="mb-2">
                <button
                  @click.prevent="
                    modalImageUrl = modalTarget.imageUrl;
                    imageOnChange = 'main';
                  "
                  class="btn btn-outline-success btn-sm d-block w-100"
                >
                  主要圖片
                </button>
              </div>
              <div v-if="modalTarget.imagesUrl" class="mb-2">
                <p class="mb-1">次要圖片</p>
                <button
                  v-for="(img, index) in modalTarget.imagesUrl"
                  :key="index"
                  @click.prevent="
                    modalImageUrl = img;
                    imageOnChange = index;
                  "
                  class="btn btn-outline-success btn-sm"
                >
                  {{ index }}
                </button>
              </div>
              <div v-if="modalTarget.imageUrl" class="mb-2">
                <button
                  @click.prevent="imageEdit()"
                  class="btn btn-outline-warning btn-sm d-block w-100"
                >
                  修改圖片
                </button>
              </div>
              <div class="mb-2">
                <button
                  @click.prevent="imageAdd()"
                  class="btn btn-outline-primary btn-sm d-block w-100"
                >
                  新增圖片
                </button>
              </div>
              <div>
                <button class="btn btn-outline-danger btn-sm d-block w-100">
                  刪除圖片
                </button>
              </div>
            </div>
            <div class="col-sm-8">
              <div class="mb-3">
                <label for="title" class="form-label">標題</label>
                <input
                  v-model="modalTarget.title"
                  id="title"
                  type="text"
                  class="form-control"
                  placeholder="請輸入標題"
                />
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="category" class="form-label">分類</label>
                  <input
                    v-model="modalTarget.category"
                    id="category"
                    type="text"
                    class="form-control"
                    placeholder="請輸入分類"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">單位</label>
                  <input
                    v-model="modalTarget.unit"
                    id="unit"
                    type="text"
                    class="form-control"
                    placeholder="請輸入單位"
                  />
                </div>
              </div>

              <div class="row">
                <div class="mb-3 col-md-6">
                  <label for="origin_price" class="form-label">原價</label>
                  <input
                    v-model="modalTarget.origin_price"
                    id="origin_price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入原價"
                  />
                </div>
                <div class="mb-3 col-md-6">
                  <label for="price" class="form-label">售價</label>
                  <input
                    v-model="modalTarget.price"
                    id="price"
                    type="number"
                    min="0"
                    class="form-control"
                    placeholder="請輸入售價"
                  />
                </div>
              </div>
              <hr />

              <div class="mb-3">
                <label for="description" class="form-label">產品描述</label>
                <textarea
                  v-model="modalTarget.content"
                  id="content"
                  type="text"
                  class="form-control"
                  placeholder="請輸入產品描述"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <label for="content" class="form-label">說明內容</label>
                <textarea
                  v-model="modalTarget.description"
                  id="description"
                  type="text"
                  class="form-control"
                  placeholder="請輸入說明內容"
                >
                </textarea>
              </div>
              <div class="mb-3">
                <div class="form-check">
                  <input
                    v-model="modalTarget.is_enabled"
                    id="is_enabled"
                    class="form-check-input"
                    type="checkbox"
                    :true-value="1"
                    :false-value="0"
                  />
                  <label class="form-check-label" for="is_enabled"
                    >是否啟用</label
                  >
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button
            @click.prevent="sendProductData()"
            type="button"
            class="btn btn-primary"
          >
            確認
          </button>
        </div>
      </div>
    </div>
  </div>
  <div
    id="delProductModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="delProductModalLabel"
    aria-hidden="true"
  >
    <div class="modal-dialog">
      <div class="modal-content border-0">
        <div class="modal-header bg-danger text-white">
          <h5 id="delProductModalLabel" class="modal-title">
            <span>刪除產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          是否刪除
          <strong class="text-danger">{{ modalTarget.title }}</strong>
          商品(刪除後將無法恢復)。
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-secondary"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button
            @click.prevent="deleteProduct()"
            type="button"
            class="btn btn-danger"
          >
            確認刪除
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
