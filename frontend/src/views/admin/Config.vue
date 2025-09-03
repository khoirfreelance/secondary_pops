<template>
  <div class="configuration-wrapper">
    <!-- Header -->
    <HeaderAdmin :is-collapsed="isCollapsed" @toggle-sidebar="toggleSidebar" />

    <div class="d-flex flex-column flex-md-row">
      <!-- Sidebar -->
      <NavbarAdmin :is-collapsed="isCollapsed" />

      <div class="flex-grow-1 d-flex flex-column">
        <!-- Gradient Banner -->
        <div class="position-relative">
          <div
            class="configuration-banner text-white p-5 d-flex flex-column flex-md-row justify-content-between align-items-center"
          >
            <!-- Left Content -->
            <div>
              <h2 class="fw-bold mb-2">Konfigurasi</h2>
              <p class="mb-0">Halaman konfigurasi untuk melakukan pengaturan administrator</p>
            </div>

            <!-- Right Content: Breadcrumb -->
            <nav aria-label="breadcrumb" class="mt-3 mt-md-0">
              <ol class="breadcrumb mb-0">
                <li class="breadcrumb-item">
                  <router-link to="/admin" class="text-decoration-none text-white-50"
                    >Beranda</router-link
                  >
                </li>
                <li class="breadcrumb-item active text-white" aria-current="page">Konfigurasi</li>
              </ol>
            </nav>
          </div>
        </div>

        <!-- Content -->
        <div
          class="flex-grow-1 p-4"
          :style="{
            backgroundImage: form.background ? `url(${form.background})` : 'none',
            backgroundSize: 'cover',
            backgroundPosition: 'center',
            backgroundAttachment: 'fixed',
          }"
        >
          <div class="card glass-card border-0 rounded-4 shadow-sm">
            <div class="card-body p-4">
              <form @submit.prevent="handleSubmit" class="row g-4">
                <!-- Upload Section -->
                <div class="col-12">
                  <h6 class="section-title">Branding</h6>
                </div>

                <!-- Upload Logo -->
                <div class="col-md-6">
                  <label class="form-label fw-semibold">Upload Logo</label>
                  <div
                    class="dropzone p-4 rounded-3 text-center"
                    :class="{ 'border-primary bg-light': isLogoDrag }"
                    @dragover.prevent="isLogoDrag = true"
                    @dragleave.prevent="isLogoDrag = false"
                    @drop.prevent="handleDrop($event, 'logo')"
                  >
                    <i class="bi bi-cloud-upload fs-1 text-primary"></i>
                    <p class="mb-1 fw-medium">Drag & drop logo here</p>
                    <small class="text-muted">or click to browse</small>
                    <input
                      type="file"
                      accept="image/*"
                      class="position-absolute w-100 h-100 top-0 start-0 opacity-0"
                      @change="handleFileChange($event, 'logo')"
                    />
                  </div>
                </div>

                <!-- Preview Logo-->
                <div class="col-md-6">
                  <label class="form-label fw-semibold">Preview</label>
                  <div v-if="previewLogo" class="mt-2 text-center">
                    <img
                      :src="previewLogo"
                      alt="Logo Preview"
                      class="img-fluid rounded shadow-sm"
                      style="max-height: 100px"
                    />
                  </div>
                  <input
                    type="number"
                    v-model="form.logoWidth"
                    class="form-control modern-input mt-3"
                    placeholder="Logo Width (px) e.g. 120"
                  />
                </div>

                <!-- Upload Background -->
                <div class="col-md-6">
                  <label class="form-label fw-semibold">Upload Background</label>
                  <div
                    class="dropzone p-4 rounded-3 text-center"
                    :class="{ 'border-primary bg-light': isBgDrag }"
                    @dragover.prevent="isBgDrag = true"
                    @dragleave.prevent="isBgDrag = false"
                    @drop.prevent="handleDrop($event, 'background')"
                  >
                    <i class="bi bi-image fs-1 text-primary"></i>
                    <p class="mb-1 fw-medium">Drag & drop background</p>
                    <small class="text-muted">or click to browse</small>
                    <input
                      type="file"
                      accept="image/*"
                      class="position-absolute w-100 h-100 top-0 start-0 opacity-0"
                      @change="handleFileChange($event, 'background')"
                    />
                  </div>
                </div>

                <!-- Preview Background-->
                <div class="col-md-6">
                  <label class="form-label fw-semibold">Preview</label>
                  <div v-if="previewBackground" class="mt-3 text-center">
                    <img
                      :src="previewBackground"
                      alt="Background Preview"
                      class="img-fluid rounded shadow-sm"
                      style="max-height: 150px"
                    />
                  </div>
                </div>

                <!-- Theme -->
                <div class="col-md-12">
                  <label class="form-label fw-semibold">Color Theme</label>
                  <input
                    type="color"
                    v-model="form.colorTheme"
                    class="form-control form-control-color modern-input"
                  />
                </div>

                <!-- Maintenance Mode -->
                <div class="col-md-12 d-flex align-items-center justify-content-between">
                  <label class="form-label fw-semibold mb-0">Maintenance Mode</label>
                  <div class="form-check form-switch">
                    <input
                      class="form-check-input"
                      type="checkbox"
                      role="switch"
                      v-model="form.maintenance"
                    />
                    <label class="form-check-label">
                      {{ form.maintenance ? 'On' : 'Off' }}
                    </label>
                  </div>
                </div>

                <!-- Save Button -->
                <div class="col-12 text-end">
                  <button type="submit" class="btn btn-primary px-5 py-2 rounded-pill shadow">
                    <i class="bi bi-save me-2"></i> Simpan
                  </button>
                </div>
              </form>
            </div>
          </div>
        </div>

        <CopyRight />
      </div>
    </div>
  </div>

  <!-- Modal Success -->
  <div class="modal fade" id="successModal" tabindex="-1" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content border-0 shadow-lg rounded-4">
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
          <p class="mb-0">Pengaturan berhasil disimpan ke <strong>localStorage</strong>.</p>
        </div>
        <div class="modal-footer justify-content-center">
          <button type="button" class="btn btn-success rounded-pill px-4" data-bs-dismiss="modal">
            OK
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.configuration-wrapper {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #f9f9fb;
  min-height: 100vh;
}

/* Gradient Banner */
.configuration-banner {
  background: linear-gradient(90deg, var(--bs-primary), var(--bs-secondary));
  border-radius: 0 0 1.5rem 1.5rem;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}

/* Card Glassmorphism */
.glass-card {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.4);
}

/* Dropzone */
.dropzone {
  position: relative;
  cursor: pointer;
  border: 2px dashed #dee2e6;
  transition: all 0.3s ease;
}
.dropzone:hover {
  border-color: var(--bs-primary);
  background: #f8faff;
  transform: translateY(-2px);
}

/* Inputs */
.modern-input {
  transition: all 0.2s ease;
}
.modern-input:focus {
  border-color: var(--bs-primary);
  box-shadow: 0 0 0 0.25rem rgba(13, 110, 253, 0.25);
}

/* Section Title */
.section-title {
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-weight: 600;
  color: #6c757d;
  border-bottom: 1px solid #dee2e6;
  padding-bottom: 0.3rem;
  margin-bottom: 1rem;
}

/* Button Gradient */
.btn-gradient {
  background: linear-gradient(135deg, #198754, var(--bs-secondary));
  border: none;
  color: white;
  transition: all 0.3s ease;
}
.btn-gradient:hover {
  background: linear-gradient(135deg, #157347, #198754);
  transform: translateY(-2px);
}
</style>

<script>
import CopyRight from '@/components/CopyRight.vue'
import HeaderAdmin from '@/components/HeaderAdmin.vue'
import NavbarAdmin from '@/components/NavbarAdmin.vue'

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Config',
  components: { NavbarAdmin, CopyRight, HeaderAdmin },
  data() {
    return {
      form: {
        logo: null,
        logoWidth: 120,
        background: null,
        colorTheme: 'var(--bs-primary)',
        footerColumn: 3,
        maintenance: false,
      },
      isCollapsed: false,
      previewLogo: null,
      previewBackground: null,
      isLogoDrag: false,
      isBgDrag: false,
    }
  },
  mounted() {
    // Load dari localStorage kalau ada
    const saved = localStorage.getItem('siteConfig')
    if (saved) {
      const parsed = JSON.parse(saved)
      this.form = parsed
      this.previewLogo = parsed.logo || null
      this.previewBackground = parsed.background || null
    }
  },
  methods: {
    toggleSidebar() {
      this.isCollapsed = !this.isCollapsed
    },
    handleFileChange(e, type) {
      const file = e.target.files[0]
      if (!file) return
      this.setFile(file, type)
    },
    handleDrop(e, type) {
      const file = e.dataTransfer.files[0]
      if (!file) return
      this.setFile(file, type)
      if (type === 'logo') this.isLogoDrag = false
      if (type === 'background') this.isBgDrag = false
    },
    setFile(file, type) {
      const reader = new FileReader()
      reader.onload = (ev) => {
        if (type === 'logo') {
          this.form.logo = ev.target.result // simpan base64
          this.previewLogo = ev.target.result
        } else if (type === 'background') {
          this.form.background = ev.target.result
          this.previewBackground = ev.target.result
        }
      }
      reader.readAsDataURL(file)
    },
    handleSubmit() {
      // Simpan ke localStorage
      localStorage.setItem('siteConfig', JSON.stringify(this.form))
      // Tampilkan modal sukses
      // eslint-disable-next-line no-undef
      const modal = new bootstrap.Modal(document.getElementById('successModal'))
      modal.show()
    },
  },
}
</script>
