<template>
  <div>
    <!--Stats cards-->
    <div class="row">
      <div class="col-md-6 col-lg-6 col-sm-12 offset-md-3 offset-lg-3">
        <p for="keyword" class="text-center">
          Masukan Kata Kunci <small><strong>( Min 3 Karakter )</strong></small>
        </p>
        <p>Total Permohonan : {{ requestLength }}</p>
        <input
          type="text"
          class="form-control border form-control-lg"
          name="keyword"
          placeholder="Email / Kode Berkas / Nama Pemohon / Nama Pemberi Kuasa"
          v-model="keyword"
        />
      </div>
    </div>

    <div class="row">
      <div class="col-md-6 col-lg-6 col-sm-12 offset-md-3 offset-lg-3">
        <div class="d-flex justify-content-center" v-if="loadingOverlay">
          <FacebookLoader :color="'#35495e'"></FacebookLoader>
        </div>
      </div>
    </div>

    <div class="row my-2" v-if="isResultExists">
      <div class="col d-flex justify-content-center">
        <h4>Tidak Terdapat Pengajuan Permohonan</h4>
      </div>
    </div>

    <div
      class="row pointer"
      v-for="item in resultList"
      :key="item.id"
      @click="openDetailRequest(item)"
    >
      <div class="col-md-8 col-lg-8 col-sm-12 offset-md-2 offset-lg-2">
        <div class="card p-2 my-2 border">
          <div
            class="d-flex justify-content-between align-items-center px-2 py-2"
          >
            <div class="information">
              <div class="d-flex mb-1">
                <span class="badge badge-success px-2 py-2">
                  {{ item.unique_id }}
                </span>
                <span class="badge badge-secondary mx-2 px-2 py-2">
                  {{ item.service.service_name }}
                </span>
              </div>
              <div class="d-flex align-items-center">
                <h4 class="m-0 mr-2">
                  {{ item.authorized_name }}
                </h4>
                <span>( {{ item.email }} )</span>
              </div>
            </div>
            <dropdown :trigger="'click'" :is-icon="false">
              <template slot="btn">
                <span class="ti-more pointer"></span>
              </template>
              <template slot="body">
                <ul class="dropdown-child">
                  <li @click="openDetailRequest(item)">
                    <span class="ti-eye mr-2"></span>
                    Detail
                  </li>
                </ul>
              </template>
            </dropdown>
          </div>
        </div>
      </div>
    </div>

    <modal-vue
      v-if="modal.displayModalDetail"
      @close="modal.displayModalDetail = false"
    >
      <template #header>
        <div class="d-flex justify-content-between align-items-center">
          <h4 class="m-0">{{ document.authorized_name }}</h4>
        </div>
      </template>
      <template #body>
        <v-loading
          :active.sync="modal.loading"
          :can-cancel="false"
          :is-full-page="false"
        ></v-loading>
        <div class="row mx-2">
          <div class="col-8">
            <label-horizontal-vue>
              <template #left-column>
                Penerima Kuasa / Pemohon
              </template>
              <template #right-column>
                {{ document.authorized_name }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue>
              <template #left-column>
                Email
              </template>
              <template #right-column>
                {{ document.email }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue>
              <template #left-column>
                Telephone Pemohon / Kuasa
              </template>
              <template #right-column>
                {{ document.authorized_phone_number }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue v-if="document.authorizer_name">
              <template #left-column>
                Nama Pemberi Kuasa
              </template>
              <template #right-column>
                {{ document.authorizer_name }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue>
              <template #left-column>
                Jenis Pelayanan
              </template>
              <template #right-column>
                {{ document.service.service_name }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue v-if="document.authorizer_name">
              <template #left-column>
                Pemberi Kuasa
              </template>
              <template #right-column>
                {{ document.authorizer_name || "-" }}
              </template>
            </label-horizontal-vue>

            <label-horizontal-vue v-if="document.authorizer_card_identity">
              <template #left-column>
                Nomor KTP Pemberi Kuasa
              </template>
              <template #right-column>
                {{ document.authorizer_card_identity || "-" }}
              </template>
            </label-horizontal-vue>
          </div>
          <div class="col-4">
            <div class="row">
              <div class="col-12">
                <textarea
                  name="description"
                  id="description"
                  cols="30"
                  rows="5"
                  class="form-control border"
                  placeholder="Keterangan"
                  v-model="modal.descriptionBox"
                ></textarea>
              </div>
              <div class="col-12 d-flex align-items-center my-2">
                <button
                  class="btn btn-danger flex-fill mx-2"
                  @click="discardRequest"
                >
                  Tolak
                </button>
                <button
                  class="btn btn-primary flex-fill mx-2"
                  @click="acceptRequest"
                >
                  Terima
                </button>
              </div>
            </div>
          </div>
        </div>

        <div class="row mx-2">
          <div class="col">
            <table class="table">
              <tbody>
                <tr>
                  <td>KTP Penerima Kuasa / Pemohon</td>
                  <td>
                    <DownloadButtonVue
                      :file="{
                        file_path: document.authorized_card_path
                      }"
                    />
                  </td>
                </tr>
                <tr v-if="document.authorizer_card_path">
                  <td>KTP Pemberi Kuasa</td>
                  <td>
                    <DownloadButtonVue
                      :file="{
                        file_path: document.authorizer_card_path
                      }"
                    />
                  </td>
                </tr>
                <tr v-for="file in document.files" :key="file.id">
                  <td>{{ file.file_type }}</td>
                  <td>
                    <DownloadButtonVue :file="file" />
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </template>
    </modal-vue>
  </div>
</template>
<script>
import { FacebookLoader } from "vue-spinners-css";
import { StatsCard, ChartCard } from "@/components/index";
import Chartist from "chartist";
import {
  apiFindDocument,
  apiGetDetailDocument,
  apiPutDocument
} from "../http/api";
import Dropdown from "bp-vuejs-dropdown";
import ModalVue from "../components/Modal.vue";
import LabelHorizontalVue from "../components/LabelHorizontal.vue";
import { downloadFileHelpers } from "../helpers/utils";
import DownloadButtonVue from "../components/DownloadButton.vue";
import { mapState } from "vuex";

export default {
  components: {
    StatsCard,
    ChartCard,
    FacebookLoader,
    Dropdown,
    ModalVue,
    LabelHorizontalVue,
    DownloadButtonVue
  },
  /**
   * Chart data used to render stats, charts. Should be replaced with server data
   */
  data() {
    return {
      keyword: null,
      loadingOverlay: false,
      statsCards: [
        {
          type: "warning",
          icon: "ti-server",
          title: "Dokumen",
          value: "8",
          footerText: "Unggah Dokumen",
          footerIcon: "ti-upload"
        }
      ],
      resultList: [],
      requestLength: 0,
      modal: {
        displayModalDetail: false,
        // detailItem: {},
        descriptionBox: null,
        loading: false
      },
      isResultExists: false
    };
  },
  watch: {
    keyword: function(value) {
      if (value.length >= 3) {
        this.loadingOverlay = true;
        this.isResultExists = false;
        apiFindDocument(value)
          .then(result => {
            this.resultList = result.data.documents;
            if (result.data.documents.length <= 0) {
              this.isResultExists = true;
            }
          })
          .catch(err => {
            console.error(err, "error");
          })
          .finally(() => {
            this.loadingOverlay = false;
          });
      } else if (value.length <= 0) {
        this.loadingOverlay = false;
        this.isResultExists = false;
        this.resultList = [];
      }
    }
  },
  methods: {
    openDetailRequest(item) {
      const { id } = item;

      this.$store
        .dispatch("apiGetDetailDocument", { doc_id: id })
        .then(result => {
          this.modal.displayModalDetail = true;
        })
        .catch(err => {
          this.$toast.error("Terjadi Kesalahan pada Server");
        });
    },
    acceptRequest() {
      this.modal.loading = true;

      apiPutDocument(this.document.id, {
        description: this.modal.descriptionBox,
        is_done: true,
        is_waiting: false
      })
        .then(result => {
          this.modal.descriptionBox = null;
          this.$toast.success("Permohonan berhasil di terima");
          return apiFindDocument();
        })
        .then(result => {
          this.requestLength = result.data.documents.length;
          if (result.data.documents.length <= 0) {
            this.isResultExists = true;
          }
        })
        .catch(err => {
          this.$toast.error("Terjadi Kesalahan pada Server");
        })
        .finally(() => {
          this.modal.loading = false;
          this.modal.displayModalDetail = false;
          this.resultList = [];
        });
    },
    discardRequest() {
      if (!this.modal.descriptionBox) {
        this.$toast.error("Harap isi keterangan Penolakan");
        return;
      }

      this.modal.loading = true;

      apiPutDocument(this.document.id, {
        description: this.modal.descriptionBox,
        is_done: false,
        is_waiting: false
      })
        .then(result => {
          this.modal.descriptionBox = null;
          this.$toast.success("Permohonan berhasil di tolak");
          return apiFindDocument();
        })
        .then(result => {
          this.requestLength = result.data.documents.length;
          if (result.data.documents.length <= 0) {
            this.isResultExists = true;
          }
        })
        .catch(err => {
          this.$toast.error("Terjadi Kesalahan pada Server");
        })
        .finally(() => {
          this.modal.loading = false;
          this.modal.displayModalDetail = false;
          this.resultList = [];
        });
    }
  },
  created() {
    apiFindDocument()
      .then(result => {
        this.requestLength = result.data.documents.length;
        if (result.data.documents.length <= 0) {
          this.isResultExists = true;
        }
      })
      .catch(err => {
        console.error(err, "error");
      })
      .finally(() => {
        this.loadingOverlay = false;
      });
  },
  computed: {
    ...mapState({ document: "detailDocument" })
  }
};
</script>
<style>
.bp-dropdown__body {
  padding: 0 !important;
  margin: 0 !important;
  border-radius: 4px;
}

.card {
  z-index: 0 !important;
}
</style>
<style lang="scss" scoped>
.pointer {
  cursor: pointer;
}

.dropdown-child {
  list-style: none;
  margin: 0;
  padding: 0;

  li {
    padding: 0.65rem 3rem 0.65rem 1rem;
    // margin: 0 1.5rem;

    &:hover {
      background-color: rgba($color: #000000, $alpha: 0.1);
      cursor: pointer;
    }
  }

  hr {
    margin: 0;
  }
}
</style>
