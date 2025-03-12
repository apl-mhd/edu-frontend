<script setup>
import { ref, onMounted } from 'vue'
import StudentCreateUpdate from '@/views/student/StudentCreateUpdate.vue'
import PaymentHistoryModal from './PaymentHistoryModal.vue'
import { useTemplateRef } from 'vue'
import { showToast } from '@/utils/toast'
import { BOverlay } from 'bootstrap-vue-3'

import axios from 'axios'

const batches = ref([])
const academicYears = ref([])
const homeTowns = ref([])
const colleges = ref([])
const paymentList = ref(null)

const isLoading = ref(true)

const blankStudent = {
  id: null,
  name: null,
  phone: null,
  gurdian_phone: null,
  email: null,
  gender: 'M',
  batch: null,
  hsc_batch: null,
  home_town: null,
  college: null,
}
const student = ref(JSON.parse(JSON.stringify(blankStudent)))

const errors = ref({})
const studentsList = ref({ results: [] })
const studentPaymentList = ref([])
const studentTableRow = ref({})

const q = ref('')
const sortBy = ref('')
const batchBy = ref('')
const filterBy = ref('')
const paymentFormData = ref({
  student: null,
  payment_amount: 0,
})

const isLoadingFalse = () => {
  isLoading.value = false
}
const isLoadingTrue = () => {
  isLoading.value = true
}

const resetStudent = () => {
  student.value = JSON.parse(JSON.stringify(blankStudent))
}

const resetForm = () => {
  resetStudent()
  errors.value = {}
}

const getAllStudent = () => {
  isLoadingFalse()
  axios
    .get(
      `http://127.0.0.1:8000/students/filter/?q=${q.value}&filter_by=${sortBy.value}${filterBy.value}&batch=${batchBy.value}`,
    )
    .then((response) => {
      studentsList.value.results = response.data
      isLoadingFalse()
    })
    .catch((error) => {})
}

const submitStudentForm = (studentData) => {
  isLoadingTrue()
  if (studentData.id) {
    axios
      .put(`http://127.0.0.1:8000/api/students/${studentData.id}/`, studentData)
      .then((response) => {
        console.log(response.data)
        getAllStudent()
        resetStudent()
        showToast("Student's information updated successfully", 'success')
        errors.value = {}
      })
      .catch((error) => {
        const errorsList = error.response.data
        const keys = Object.keys(errorsList)

        keys.forEach((i) => {
          showToast(`${i}: ${errorsList[i][0]}`, 'warning')
          errors.value[i] = errorsList[i][0]
        })
        isLoadingFalse()
      })
  } else {
    axios
      .post('http://127.0.0.1:8000/api/students/create/', studentData)
      .then((response) => {
        showToast(response.data.message, 'success')
        resetStudent()
        errors.value = {}
        getAllStudent()
      })
      .catch((error) => {
        const errorsList = error.response.data.errors
        const keys = Object.keys(errorsList)

        keys.forEach((i) => {
          showToast(`${i}: ${errorsList[i][0]}`, 'warning')
          errors.value[i] = errorsList[i][0]
        })
        isLoadingFalse()
      })
  }
}

const getStudent = (studentId) => {
  isLoading.value = true
  axios
    .get(`http://127.0.0.1:8000/api/students/${studentId}/`)
    .then((response) => {
      student.value = { ...response.data }
      isLoading.value = false
    })
    .catch((error) => {})
}

const getStudenDataForm = () => {
  axios.get('http://127.0.0.1:8000/api/students/data-form/').then((response) => {
    batches.value = response.data.batches
    academicYears.value = response.data.academic_years
    homeTowns.value = response.data.home_towns
    colleges.value = response.data.colleges
  })
}

//get payment history

const getPaymentHistory = (student) => {
  studentTableRow.value = student
  axios
    .get(`http://127.0.0.1:8000/course/student-payment-list/${student.id}/`)
    .then((response) => {
      studentPaymentList.value = response.data.results
      openModalRef.value.openModal()
    })
    .catch((error) => {
      const errorData = error.response.data
      let msg = ''
      // for (const [key, value] of Object.entries(errorData.errors)) {
      //   msg += `${key}: ${value} <br>`
      // }

      // this.showToast(msg, 'error')
    })
}

onMounted(() => {
  getAllStudent()
  getStudenDataForm()
})

const openModalRef = useTemplateRef('openModalRef')
</script>

<template>
  <b-overlay :show="isLoading">
    <div></div>
    {{ student }}
    <student-create-update
      v-model:student="student"
      :batches="batches"
      :academic-years="academicYears"
      :home-towns="homeTowns"
      :colleges="colleges"
      :errors="errors"
      @submit-student-form="submitStudentForm"
      @reset-form="resetForm"
    />
    <div class="mb-4"></div>

    <div>
      <!--Start Student Table-->
      <div class="row mt-5">
        <div>
          <table class="table table-light table-striped table-hover table-bordered table-sm">
            <thead class="table-dark">
              <tr>
                <th scope="col">ID</th>
                <th scope="col">Student Info</th>
                <th scope="col">Batch</th>
                <th scope="col">HSC Batch</th>
                <th scope="col">Due Amount</th>
                <th scope="col">Paid This Month</th>
                <th scope="col">Action</th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for="(i, index) in studentsList.results"
                :key="i.id"
                :class="i.paid_current_month ? 'table-success' : 'table-danger'"
              >
                <td scope="row">{{ index + 1 }}</td>

                <td>
                  <small>{{ i.name }}</small> <br />
                  <small>Phone: {{ i.phone }}</small> <br />
                  <small>ID# {{ i.student_roll }}</small> <br />
                </td>
                <td>
                  <small class="">{{ i.batch__name }}</small> <br />
                  <small>{{ i.batch__start_time }} - {{ i.batch__end_time }}</small>
                </td>
                <td>{{ i.hsc_batch__year }} - {{ i.hsc_batch__year + 1 }}</td>
                <td>{{ i.due_amount ? i.due_amount : 0 }}</td>
                <td>
                  <span
                    class="me-2"
                    v-html="
                      i.paid_current_month
                        ? `<span class='badge text-bg-primary'>Yes</span>`
                        : `<span class='badge text-bg-danger'>No</span>`
                    "
                  ></span>

                  <span class="badge ml-3 text-bg-info" @click="getPaymentHistory(i)"
                    >Payment History</span
                  ><br />
                  <small>{{ i.latest_payment }}</small>
                </td>
                <td>
                  <div class="btn-group" role="group">
                    <button
                      type="button"
                      class="btn btn-primary dropdown-toggle btn-sm"
                      data-bs-toggle="dropdown"
                      aria-expanded="false"
                    >
                      Action
                    </button>
                    <ul class="dropdown-menu">
                      <li>
                        <a
                          href="#"
                          class="dropdown-item"
                          data-bs-toggle="modal"
                          data-bs-target="#paymentModal"
                          @click="studentData(i)"
                          >Make Payment</a
                        >
                      </li>
                      <hr />
                      <li>
                        <a
                          href="#"
                          data-bs-toggle="modal"
                          data-bs-target="#assignCourseModal"
                          class="dropdown-item"
                          @click="studentData(i)"
                          >Assign Course</a
                        >
                      </li>
                      <hr />
                      <li>
                        <a href="#" class="dropdown-item">Assign Material</a>
                      </li>
                      <hr />
                      <li>
                        <a href="#" class="dropdown-item" @click="getStudent(i.id)">Edit</a>
                      </li>
                    </ul>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <nav aria-label="Page navigation example">
          <ul class="pagination">
            <li class="page-item">
              <a class="page-link" :href="studentsList.prev">Previous</a>
            </li>
            <li class="page-item"><a class="page-link" :href="studentsList.next">Next</a></li>
          </ul>
        </nav>
      </div>
    </div>

    <!-- modal payment history -->
    <payment-history-modal
      ref="openModalRef"
      :student-payment-list="studentPaymentList"
      :student-table-row="studentTableRow"
    />

    <!--Start Make Payment modal-->
    <div
      class="modal fade"
      id="paymentModal"
      tabindex="-1"
      aria-labelledby="paymentModalLable"
      aria-hidden="true"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title fs-5" id="paymentModalLable">
              Payment of ID# [[studentTableRow.student_roll]] --- Name: [[studentTableRow.name]]
            </h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="submitPaymentForm">
              <input type="text" hidden v-model="paymentFormData.student" />
              <div class="row mb-3">
                <div class="col">
                  <label for="courseFee" class="col-form-label">Due Amount</label>
                  <input
                    type="text"
                    class="form-control"
                    id="courseFee"
                    disabled
                    :value="studentTableRow.due_amount - paymentFormData.payment_amount"
                  />
                </div>
                <div class="col">
                  <label for="payment" class="col-form-label">Payment Amount</label>
                  <input
                    min="1"
                    type="text"
                    class="form-control"
                    v-model="paymentFormData.payment_amount"
                    id="payment"
                  />
                </div>
              </div>

              <div class="mb-3">
                <button type="button" class="btn btn-danger" data-bs-dismiss="modal">Close</button>
                <button type="submit" class="btn btn-primary" data-bs-dismiss="modal">
                  Pay Now
                </button>
              </div>
            </form>
          </div>
          <div class="footer"></div>
        </div>
      </div>
    </div>
    <!--End Payment Modal-->
  </b-overlay>
</template>

<style scoped></style>
