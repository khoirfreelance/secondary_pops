<template>
  <div class="family-wrapper">
    <!-- Header -->
    <HeaderAdmin :is-collapsed="isCollapsed" @toggle-sidebar="toggleSidebar" />

    <div class="d-flex flex-column flex-md-row">
      <!-- Sidebar -->
      <NavbarAdmin :is-collapsed="isCollapsed" />

      <!-- Main Content -->
      <div
        class="flex-grow-1 d-flex flex-column"
        :style="{
          backgroundImage: background ? `url(${background})` : 'none',
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          backgroundAttachment: 'fixed',
        }"
      >
        <!-- Banner -->
        <div
          class="family-banner text-white p-5 d-flex flex-column flex-md-row justify-content-between align-items-center"
        >
          <div>
            <h2 class="fw-bold mb-2">Data Keluarga</h2>
            <p class="mb-0">List daftar keluarga yang terdaftar di dalam posyandu</p>
          </div>
          <nav aria-label="breadcrumb" class="mt-3 mt-md-0">
            <ol class="breadcrumb mb-0">
              <li class="breadcrumb-item">
                <router-link to="/admin" class="text-decoration-none text-white-50">
                  Beranda
                </router-link>
              </li>
              <li class="breadcrumb-item active text-white" aria-current="page">Data Keluarga</li>
            </ol>
          </nav>
        </div>

        <!-- Filter -->
        <div class="filter-wrapper bg-light rounded shadow-sm p-3 mt-3 container-fluid">
          <form class="row g-3 align-items-end" @submit.prevent="applyFilter">
            <!-- NIK (selalu tampil, realtime filter) -->
            <div class="col-md-6">
              <label for="nik" class="form-label">NIK</label>
              <input
                type="text"
                v-model="filter.nik"
                id="nik"
                class="form-control"
                placeholder="Cari berdasarkan NIK"
              />
            </div>
            <div class="col-md-6">
              <label for="no_kk" class="form-label">No. Kartu Keluarga</label>
              <input
                type="text"
                v-model="filter.no_kk"
                id="no_kk"
                class="form-control"
                placeholder="Cari berdasarkan No. Kartu Keluarga"
              />
            </div>

            <!-- Expandable section -->
            <div v-if="isFilterOpen" class="row g-3 align-items-end mt-2">
              <!-- Nama -->
              <div class="col-md-6">
                <label for="kepala" class="form-label">Nama Kepala Keluarga</label>
                <input
                  type="text"
                  v-model="advancedFilter.kepala"
                  id="kepala"
                  class="form-control"
                />
              </div>

              <!-- RT -->
              <div class="col-md-3">
                <label for="rt" class="form-label">RT</label>
                <input type="number" v-model="advancedFilter.rt" id="rt" class="form-control" />
              </div>

              <!-- RW -->
              <div class="col-md-3">
                <label for="rw" class="form-label">RW</label>
                <input type="number" v-model="advancedFilter.rw" id="rw" class="form-control" />
              </div>

              <!-- Tombol -->
              <div class="col-md-12">
                <button type="submit" class="btn btn-primary float-start" @click="applyFilter">
                  <i class="bi bi-search"></i> Cari
                </button>
                <button type="button" class="btn btn-secondary float-end" @click="resetFilter">
                  <i class="bi bi-arrow-clockwise"></i> Reset
                </button>
              </div>
            </div>
          </form>

          <!-- Expand/Collapse Button -->
          <div class="text-end mt-2">
            <button type="button" class="btn btn-outline-secondary btn-sm" @click="toggleExpand">
              <i :class="isFilterOpen ? 'bi bi-chevron-up' : 'bi bi-chevron-down'"></i>
              {{ isFilterOpen ? 'Tutup Filter Lain' : 'Filter Lain' }}
            </button>
          </div>
        </div>

        <!-- Button Group -->
        <div class="container-fluid mt-4 d-flex flex-wrap gap-2 justify-content-end">
          <button class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modalTambah">
            <i class="bi bi-plus-square"></i> Tambah Data
          </button>
          <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#modalImport">
            <i class="bi bi-filetype-csv"></i> Import Data Keluarga
          </button>
        </div>

        <!-- Alert -->
        <div class="container-fluid mt-4">
          <div class="alert alert-success shadow-sm">✅ Data berhasil disimpan!</div>
        </div>

        <!-- Table -->
        <div class="container-fluid">
          <div class="card modern-card mt-4">
            <div class="card-body">
              <div class="table-responsive">
                <EasyDataTable
                  :headers="headers"
                  :items="filteredFamily"
                  buttons-pagination
                  :rows-per-page="5"
                  table-class="table-modern"
                  theme-color="var(--bs-primary)"
                />
              </div>
            </div>
          </div>
        </div>

        <CopyRight class="mt-auto" />
      </div>
    </div>
  </div>

  <!-- Modal Tambah -->
  <div class="modal fade" id="modalTambah" tabindex="-1">
    <div class="modal-dialog modal-lg modal-dialog-centered">
      <div
        class="modal-content shadow-lg border-0 rounded-4"
        :style="{
          backgroundImage: background ? `url(${background})` : 'none',
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          backgroundAttachment: 'fixed',
        }"
      >
        <!-- Header -->
        <div class="modal-header text-primary bg-light border-0 rounded-top-4">
          <h5 class="modal-title fw-bold text-primary">Tambah Data Keluarga</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>

        <!-- Body -->
        <div class="modal-body">
          <form class="row g-4">
            <!-- No KK -->
            <div class="col-md-12">
              <label class="form-label small fw-semibold text-secondary">No KK</label>
              <input
                type="text"
                class="form-control shadow-sm"
                v-model="form.no_kk"
                @input="form.no_kk = form.no_kk.replace(/\D/g, '')"
                maxlength="16"
              />
            </div>

            <!-- Alamat -->
            <div class="col-md-12">
              <label class="form-label small fw-semibold text-secondary">Alamat</label>
              <textarea class="form-control shadow-sm" rows="2" v-model="form.alamat"></textarea>
            </div>

            <!-- RT, RW , Kelurahan -->
            <div class="col-md-4">
              <label class="form-label small fw-semibold text-secondary">RT</label>
              <input type="number" min="0" class="form-control shadow-sm" v-model="form.rt" />
            </div>
            <div class="col-md-4">
              <label class="form-label small fw-semibold text-secondary">RW</label>
              <input type="number" min="0" class="form-control shadow-sm" v-model="form.rw" />
            </div>
            <div class="col-md-4">
              <label class="form-label small fw-semibold text-secondary">Kelurahan</label>
              <select class="form-select shadow-sm" v-model="form.kelurahan">
                <option value="">Pilih</option>
              </select>
            </div>

            <!-- Kecamatan & Kota -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Kecamatan</label>
              <select class="form-select shadow-sm" v-model="form.kecamatan">
                <option value="">Pilih</option>
              </select>
            </div>
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Kota</label>
              <select class="form-select shadow-sm" v-model="form.kota">
                <option value="">Pilih</option>
              </select>
            </div>

            <!-- Kepala Keluarga -->
            <div class="col-12">
              <h5 class="fw-bold text-primary mt-3">Data Kepala Keluarga</h5>
            </div>

            <div class="col-md-12">
              <label class="form-label small fw-semibold text-secondary">Nama Lengkap</label>
              <input type="text" class="form-control shadow-sm" v-model="form.kepala" />
            </div>

            <!-- NIK -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">NIK</label>
              <input
                type="text"
                class="form-control shadow-sm"
                v-model="form.nik"
                maxlength="16"
                @input="form.nik = form.nik.replace(/\D/g, '')"
              />
            </div>

            <!-- Jenis Kelamin -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Jenis Kelamin</label>
              <select class="form-select shadow-sm" v-model="form.gender">
                <option value="">L/P</option>
                <option value="L">Laki-laki</option>
                <option value="P">Perempuan</option>
              </select>
            </div>

            <!-- Tempat Lahir -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Tempat Lahir</label>
              <input
                type="text"
                class="form-control shadow-sm"
                v-model="form.tmpt_lahir"
                maxlength="16"
                @input="form.tmpt_lahir = form.tmpt_lahir"
              />
            </div>

            <!-- Tanggal lahir-->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Tanggal Lahir</label>
              <input type="date" class="form-control shadow-sm" v-model="form.tgl_lahir" />
            </div>

            <!-- Pendidikan -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Pendidikan</label>
              <select class="form-select shadow-sm" v-model="form.pendidikan">
                <option value="">Pendidikan</option>
                <option value="TK">TK</option>
                <option value="SD">SD</option>
                <option value="SMP">SMP</option>
                <option value="SMA/SMK">SMA/sederajat</option>
                <option value="SARJANA">Sarjana</option>
                <option value="MAGISTER">Magister</option>
                <option value="DOKTORAL">Doktoral</option>
                <option value="PROFESSOR">Professor</option>
              </select>
            </div>

            <!-- Pekerjaan -->
            <div class="col-md-6">
              <label class="form-label small fw-semibold text-secondary">Pekerjaan</label>
              <input type="text" class="form-control shadow-sm" v-model="form.pekerjaan" />
            </div>
          </form>
        </div>

        <!-- Footer -->
        <div class="modal-footer border-0 d-flex justify-content-between">
          <button class="btn btn-light border rounded-pill px-4" data-bs-dismiss="modal">
            <i class="bi bi-x-circle me-2"></i> Batal
          </button>
          <button class="btn btn-success rounded-pill px-4" @click="saveData">
            <i class="bi bi-save me-2"></i> Simpan
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal Import -->
  <div class="modal fade" id="modalImport" ref="modalImport" tabindex="-1">
    <div class="modal-dialog">
      <div
        class="modal-content"
        :style="{
          backgroundImage: background ? `url(${background})` : 'none',
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          backgroundAttachment: 'fixed',
        }"
      >
        <div class="modal-header text-primary bg-light border-0 rounded-top-4">
          <h5 class="modal-title">Import File Ibu Hamil</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <div class="alert alert-warning p-2">
            <ul>
              <li>Import data untuk kunjungan kehamilan oleh pendampingan TPK</li>
              <li>Pastikan data yang diimport, berformat csv</li>
              <li>Pastikan data sudah lengkap sebelum di import</li>
            </ul>
          </div>
          <input type="file" class="form-control" ref="csvFile" accept=".csv" />
        </div>
        <div class="modal-footer border-0 d-flex justify-content-between">
          <button class="btn btn-light border rounded-pill px-4" data-bs-dismiss="modal">
            <i class="bi bi-x-circle me-2"></i> Batal
          </button>
          <button class="btn btn-success rounded-pill px-4" @click="handleImport">
            <i class="bi bi-upload me-2"></i> Unggah
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal Success -->
  <div class="modal fade" id="successModal" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div
        class="modal-content border-0 shadow-lg rounded-4"
        :style="{
          backgroundImage: background ? `url(${background})` : 'none',
          backgroundSize: 'cover',
          backgroundPosition: 'center',
          backgroundAttachment: 'fixed',
        }"
      >
        <div class="modal-header bg-success text-white rounded-top-4">
          <h5 class="modal-title">✅ Berhasil</h5>
          <button
            type="button"
            class="btn-close btn-close-white"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body text-center">
          <p class="mb-0">Data Anak berhasil disimpan ke <strong>localStorage</strong>.</p>
        </div>
        <div class="modal-footer justify-content-center">
          <button type="button" class="btn btn-success rounded-pill px-4" data-bs-dismiss="modal">
            OK
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Loader Overlay with Animated Progress -->
  <div
    v-if="isLoadingImport"
    class="position-fixed top-0 start-0 w-100 h-100 d-flex flex-column align-items-center justify-content-center bg-dark bg-opacity-50"
    style="z-index: 2000"
  >
    <div class="w-50">
      <div class="progress" style="height: 1.8rem; border-radius: 1rem; overflow: hidden">
        <div
          class="progress-bar progress-bar-striped progress-bar-animated"
          role="progressbar"
          :style="{ width: importProgress + '%' }"
          :data-progress="progressLevel"
        >
          <span class="fw-bold">{{ animatedProgress }}%</span>
        </div>
      </div>
    </div>
    <p class="text-white mt-3">Mengimpor data... {{ currentRow }}/{{ totalRows }} baris</p>
  </div>
</template>

<script>
import CopyRight from '@/components/CopyRight.vue'
import HeaderAdmin from '@/components/HeaderAdmin.vue'
import NavbarAdmin from '@/components/NavbarAdmin.vue'
import EasyDataTable from 'vue3-easy-data-table'
import 'vue3-easy-data-table/dist/style.css'
import { Modal } from 'bootstrap'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Family',
  components: { CopyRight, NavbarAdmin, HeaderAdmin, EasyDataTable },
  data() {
    return {
      isCollapsed: false,
      isFilterOpen: false,
      importTitle: 'Import File',
      showAlert: false,
      isLoadingImport: false,
      importProgress: 0,
      animatedProgress: 0,
      currentRow: 0,
      totalRows: 1, // default 1 agar tidak bagi 0
      form: {
        nik: '',
        no_kk: '',
        kepala: '',
        alamat: '',
        rt: '',
        rw: '',
        kelurahan: '',
        kecamatan: '',
        kota: '',
        tmpt_lahir: '',
        gender: '',
        tgl_lahir: '',
        pendidikan: '',
        pekerjaan: '',
      },
      family: [
        {
          nik: '1234567890987654',
          no_kk: '3329143112210010',
          kepala: 'Rajiah',
          rt: '3',
          rw: '13',
          tgl_lahir: '1956-08-16',
          pendidikan: 'Tamat SD/Sederajat',
        },
        {
          nik: '1234567890324564',
          no_kk: '3329143112210021',
          kepala: 'Moh. Fajar Rizqi',
          rt: '4',
          rw: '1',
          tgl_lahir: '1996-11-17',
          pendidikan: 'Diploma IV/Strata I',
        },
        {
          nik: '9334567890987654',
          no_kk: '3329143112150019',
          kepala: 'Dede Imam',
          rt: '2',
          rw: '9',
          tgl_lahir: '1991-08-15',
          pendidikan: 'Tamat SD/Sederajat',
        },
        {
          nik: '1234567453987654',
          no_kk: '3329143112150018',
          kepala: 'Nurjanah',
          rt: '3',
          rw: '9',
          tgl_lahir: '1987-09-01',
          pendidikan: 'Tamat SD/Sederajat',
        },
      ],
      headers: [
        { text: 'NIK', value: 'nik' },
        { text: 'No KK', value: 'no_kk' },
        { text: 'Kepala Keluarga', value: 'kepala' },
        { text: 'RT', value: 'rt' },
        { text: 'RW', value: 'rw' },
        { text: 'Tanggal Lahir', value: 'tgl_lahir' },
        { text: 'Pendidikan', value: 'pendidikan' },
      ],
      // filter
      filter: {
        nik: '',
        no_kk: '',
      },
      advancedFilter: {
        kepala: '',
        rt: '',
        rw: '',
      },
      appliedFilter: {}, // hasil filter simpan di sini
    }
  },
  computed: {
    background() {
      try {
        const config = JSON.parse(localStorage.getItem('siteConfig'))
        return config?.background || null
      } catch {
        return null
      }
    },
    filteredFamily() {
      return this.family.filter((item) => {
        return (
          // NIK realtime
          (!this.filter.nik || item.nik.includes(this.filter.nik)) &&
          (!this.filter.no_kk || item.no_kk.includes(this.filter.no_kk)) &&
          // Advanced filter hanya aktif setelah "Cari"
          (!this.appliedFilter.kepala ||
            item.kepala.toLowerCase().includes(this.appliedFilter.kepala.toLowerCase())) &&
          (!this.appliedFilter.rt || Number(item.rt) === Number(this.appliedFilter.rt)) &&
          (!this.appliedFilter.rw || Number(item.rw) === Number(this.appliedFilter.rw))
        )
      })
    },
  },
  methods: {
    closeModal(id) {
      const el = document.getElementById(id)
      if (el) {
        const instance = Modal.getOrCreateInstance(el)
        instance.hide()
      }

      // jaga-jaga kalau backdrop masih nyangkut
      setTimeout(() => {
        document.querySelectorAll('.modal-backdrop').forEach((el) => el.remove())
        document.body.classList.remove('modal-open')
        document.body.style.removeProperty('overflow')
        document.body.style.removeProperty('padding-right')
      }, 300) // delay biar nunggu animasi fade
    },
    updateProgressBar(percent, row, total) {
      this.importProgress = percent
      this.currentRow = row
      this.totalRows = total

      const start = this.animatedProgress
      const end = percent
      const step = (end - start) / 10
      let i = 0
      const interval = setInterval(() => {
        this.animatedProgress = Math.min(end, Math.round(start + step * i))
        i++
        if (this.animatedProgress >= end) clearInterval(interval)
      }, 30)
    },
    toggleExpand() {
      this.isFilterOpen = !this.isFilterOpen
    },
    applyFilter() {
      // copy isi advancedFilter ke appliedFilter saat tombol Cari ditekan
      this.appliedFilter = { ...this.advancedFilter }
    },
    resetFilter() {
      this.filter.nik = ''
      this.filter.no_kk = ''
      this.advancedFilter = {
        kepala: '',
        rt: '',
        rw: '',
      }
      this.appliedFilter = {}
    },
    toggleSidebar() {
      this.isCollapsed = !this.isCollapsed
    },
    saveData() {
      this.closeModal('modalTambah')
      this.isLoadingImport = true
      this.importProgress = 0
      this.animatedProgress = 0
      this.currentRow = 0
      this.totalRows = 1 // hanya 1 record, bisa disesuaikan kalau batch

      // simulasi progress bertahap
      let step = 0
      const interval = setInterval(() => {
        step += 10
        this.importProgress = Math.min(step, 100)
        this.animatedProgress = this.importProgress
        this.currentRow = Math.round((this.totalRows * this.importProgress) / 100)

        if (this.importProgress >= 100) {
          clearInterval(interval)

          // lanjut simpan data

          this.family.push({ ...this.form })
          this.showAlert = true
          setTimeout(() => (this.showAlert = false), 3000)

          // reset form
          this.form = {
            nik: '',
            no_kk: '',
            kepala: '',
            alamat: '',
            rt: '',
            rw: '',
            kelurahan: '',
            kecamatan: '',
            kota: '',
            tmpt_lahir: '',
            gender: '',
            tgl_lahir: '',
            pendidikan: '',
            pekerjaan: '',
          }

          this.$nextTick(() => {
            const el = document.getElementById('successModal')
            if (el) {
              const instance = Modal.getOrCreateInstance(el)
              instance.show()
            }
          })

          this.isLoadingImport = false
        }
      }, 150) // jeda antar progress
    },
    handleImport() {
      this.closeModal('modalImport')

      const fileInput = this.$refs.csvFile
      if (!fileInput || !fileInput.files.length) return

      this.isLoadingImport = true
      this.importProgress = 0
      this.animatedProgress = 0

      const file = fileInput.files[0]
      const reader = new FileReader()

      reader.onload = (e) => {
        const text = e.target.result
        const rows = text
          .split('\n')
          .map((r) => r.trim())
          .filter((r) => r)
        const headers = rows[0].split(',').map((h) => h.trim())
        const total = rows.length - 1
        this.totalRows = total

        rows.slice(1).forEach((row, idx) => {
          const values = row.split(',').map((v) => v.trim())
          const obj = {}
          headers.forEach((h, i) => {
            obj[h] = values[i] || ''
          })

          this.family.push({
            nik: obj.nik || '',
            no_kk: obj.no_kk || '',
            kepala: obj.kepala || '',
            tgl_lahir: obj.tgl_lahir || '',
            rt: obj.rt || '',
            rw: obj.rw || '',
            pendidikan: obj.pendidikan || '',
            alamat: obj.alamat || '',
            kelurahan: obj.kelurahan || '',
            kecamatan: obj.kecamatan || '',
            kota: obj.kota || '',
            tmpt_lahir: obj.tmpt_lahir || '',
            gender: obj.gender || '',
            pekerjaan: obj.pekerjaan || '',
          })

          const percent = Math.round(((idx + 1) / total) * 100)
          this.updateProgressBar(percent, idx + 1, total)
        })

        setTimeout(() => {
          this.isLoadingImport = false
          const el = document.getElementById('successModal')
          const instance = Modal.getOrCreateInstance(el)
          instance.show()
        }, 500)
      }

      reader.readAsText(file)
    },
  },
}
</script>

<style scoped>
.family-wrapper {
   /* tinggi navbar bootstrap default */
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f9f9fb;
  min-height: 100vh;
}
/* Gradient Banner */
.family-banner {
  background: linear-gradient(90deg, var(--bs-primary), var(--bs-secondary));
  border-radius: 0 0 1rem 1rem;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
.filter-wrapper {
  position: relative; /* biar ikut alur layout */
  z-index: 0; /* pastikan di bawah sidebar */
  margin-top: -30px !important;
  width: 97%;
  border-radius: 0.75rem;
}
/* Hilangkan garis pemisah antara sidebar dan content */
.flex-grow-1 {
  border-left: none !important;
  background-color: #f9f9fb;
}
.breadcrumb-item + .breadcrumb-item::before {
  color: rgba(255, 255, 255, 0.7);
}
/* Smooth Apple-like Modal */
.modern-modal {
  border-radius: 1.5rem;
  border: 1px solid #eaeaea;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
  background: #fff;
  transition: all 0.3s ease-in-out;
}

/* Form lebih clean */
.form-control-modern,
.form-select.form-control-modern {
  border-radius: 0.75rem;
  border: 1px solid #ddd;
  padding: 0.6rem 1rem;
  transition:
    border-color 0.2s,
    box-shadow 0.2s;
}

.form-control-modern:focus {
  border-color: var(--bs-primary);
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.2);
}

/* Animasi modal lebih halus */
.modal.fade .modal-dialog {
  transform: translateY(20px);
  transition:
    transform 0.3s ease-out,
    opacity 0.3s ease-out;
}

.modal.fade.show .modal-dialog {
  transform: translateY(0);
  opacity: 1;
}
.modern-card {
  border-radius: 1rem;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
  border: none;
}
/* Header */
.table-modern th {
  background-color: var(--bs-primary) !important; /* primary */
  color: #fff !important;
  font-weight: 600;
  padding: 0.75rem;
  text-align: left;
}

/* Cell */
.table-modern td {
  vertical-align: middle;
  padding: 0.65rem 0.75rem;
  border-bottom: 1px solid #f1f1f1;
}

/* Row hover */
.table-modern tr:hover {
  background-color: rgba(13, 110, 253, 0.08) !important;
  transition: background 0.2s ease-in-out;
}

/* Pagination & footer */
.table-modern .pagination {
  margin-top: 1rem;
}

.table-modern .pagination .page-link {
  border-radius: 0.5rem;
  color: var(--bs-primary);
}

.table-modern .pagination .active .page-link {
  background-color: #6c757d; /* secondary */
  border-color: #6c757d;
  color: #fff;
}
</style>
