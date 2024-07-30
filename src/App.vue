<template>
  <div id="app" class="d-flex justify-content-center align-items-center vh-100 position-relative">
    <div class="card p-4" style="width: 600px;">
      <h2 class="card-title text-center mb-4">Digital Respons-Ability Analytics</h2>
      <form @submit.prevent="submitForm">
        <div class="form-group" :class="{'has-error': !apiUrl}">
          <label for="api-url">API URL:</label>
          <input id="api-url" v-model="apiUrl" type="text" class="form-control" placeholder="Enter API URL" required>
        </div>
        <div class="form-group" :class="{'has-error': !apiKey}">
          <label for="api-key">API Key:</label>
          <input id="api-key" v-model="apiKey" type="text" class="form-control" placeholder="Enter API Key" required>
        </div>
        <div class="form-group" :class="{'has-error': !courseId}">
          <label for="course-id">Course ID:</label>
          <input id="course-id" v-model.number="courseId" type="number" class="form-control" placeholder="Enter Course ID" required>
        </div>
        <div class="text-center">
          <button type="submit" class="btn btn-primary mx-2" :disabled="loading">
            Generate Course Completion Report
          </button>
          <button type="button" @click="generateEnrollmentReport" class="btn btn-primary mx-2" :disabled="loading">
            Generate Enrollment Report
          </button>
        </div>
      </form>

      <!-- Bootstrap Modal for generating report -->
      <div class="custom-backdrop" v-if="loading"></div>
      <div class="modal custom-modal" id="reportModal" tabindex="-1" role="dialog" aria-labelledby="reportModalLabel" aria-hidden="true" ref="reportModal">
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-body text-center">
              <p>Generating Report...</p>
              <span class="fas fa-spinner fa-spin"></span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap';

export default {
  name: 'App',
  data() {
    return {
      apiUrl: 'https://utah.instructure.com/api/v1', // Default API URL
      apiKey: '', // Default API key (replace with actual key)
      courseId: null, // Default course ID (replace with actual ID)
      loading: false // Loading state for modal
    };
  },
  methods: {
    submitForm() {
      if (this.validateForm()) {
        this.showModal();
        this.generateReport('get_course_completion_report', 'course_completion_report');
      }
    },
    generateEnrollmentReport() {
      if (this.validateForm()) {
        this.showModal();
        this.generateReport('get_enrollment_report', 'enrollment_report');
      }
    },
    generateReport(reportType, reportName) {
      const url = `https://DigitalResponsAbility.pythonanywhere.com/${reportType}?api_url=${encodeURIComponent(this.apiUrl)}&api_key=${encodeURIComponent(this.apiKey)}&course_id=${this.courseId}`;

      // Make API request to get file
      fetch(url)
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.blob(); // Get the response as a Blob
        })
        .then(blob => {
          this.hideModal();
          const url = window.URL.createObjectURL(new Blob([blob]));
          const link = document.createElement('a');
          link.href = url;
          link.setAttribute('download', `${reportName}.xlsx`);
          document.body.appendChild(link);
          link.click();
          link.parentNode.removeChild(link);
        })
        .catch(error => {
          console.error('Error fetching file:', error);
          alert('Error generating report. Please try again.');
          this.hideModal();
        });
    },
    validateForm() {
      if (!this.apiUrl || !this.apiKey || !this.courseId) {
        alert('Please fill in all fields.');
        return false;
      }
      return true;
    },
    showModal() {
      this.loading = true;
      const modal = this.$refs.reportModal;
      modal.classList.add('show');
      modal.style.display = 'block';
    },
    hideModal() {
      this.loading = false;
      const modal = this.$refs.reportModal;
      modal.classList.remove('show');
      modal.style.display = 'none';
    }
  }
};
</script>

<style>
#app {
  font-family: 'Arial', sans-serif;
}

.card {
  max-width: 600px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.card-title {
  font-size: 1.5rem;
  font-weight: bold;
}

.form-group {
  margin-bottom: 15px;
}

.btn-primary {
  background-color: #007bff;
  border-color: #007bff;
}

.btn-primary:hover {
  background-color: #0069d9;
  border-color: #0062cc;
}

.btn-primary:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

.btn-primary:focus, .btn-primary.focus {
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.5);
}

.modal-body {
  padding: 20px 0;
}

/* Custom modal styles */
.custom-modal {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin: auto;
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 1050; /* Bootstrap modal z-index */
}

.custom-modal.show {
  display: flex;
}

.custom-backdrop {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1040; /* Bootstrap modal backdrop z-index */
}
</style>
