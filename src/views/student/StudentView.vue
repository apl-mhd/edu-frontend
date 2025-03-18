<script setup>
import { ref, onMounted } from 'vue'
import StudentCreateUpdate from '@/views/student/StudentCreateUpdate.vue'
import PaymentHistoryModal from './PaymentHistoryModal.vue'
import PaymentModal from './PaymentModal.vue'
import { useTemplateRef } from 'vue'
import { showToast } from '@/utils/toast'
import { BOverlay, BCollapse, BButton } from 'bootstrap-vue-3'

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

const blankPaymentForm = ref({
  student: null,
  payment_amount: 0,
})

const paymentFormData = ref({
  student: null,
  payment_amount: 0,
})

const filters = ref([
  {
    id: 'name',
    value: 'Name',
  },
  {
    id: 'batch__name',
    value: 'Batch',
  },
  {
    id: 'hsc_batch__year',
    value: 'HSC',
  },
  {
    id: 'due_amount',
    value: 'Due amount',
  },
  {
    id: 'paid_current_month',
    value: 'Paid',
  },
  {
    id: 'latest_payment',
    value: 'Latest payment',
  },
])

const visible = ref(false)

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

const studentData = (student) => {
  studentTableRow.value = student
  paymentFormData.value.student = student.id
  paymentModalRef.value.openModal()
  // courseAssignFormData.value.student = student.id;
}

const getStudent = (studentId) => {
  isLoading.value = true
  axios
    .get(`http://127.0.0.1:8000/api/students/${studentId}/`)
    .then((response) => {
      student.value = { ...response.data }
      isLoading.value = false
      visible.value = true
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

const submitPaymentForm = () => {
  axios
    .post('http://127.0.0.1:8000/course/payment/', paymentFormData.value)
    .then((response) => {
      paymentFormData.value = { ...blankPaymentForm.value }
      showToast(response.data.message, 'success')
      // paymentModalRef.value.closeModal()
      //getAllStudent()
      // this.showToast(response.data.message)
    })
    .catch((error) => {
      const errorData = error.response.data
    })
}

//get payment history

const getPaymentHistory = (student) => {
  studentTableRow.value = student
  axios
    .get(`http://127.0.0.1:8000/course/student-payment-list/${student.id}/`)
    .then((response) => {
      studentPaymentList.value = response.data
      paymentHistoryModalRef.value.openModal()
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

const paymentHistoryModalRef = useTemplateRef('paymentHistoryModalRef')
const paymentModalRef = useTemplateRef('paymentModalRef')
</script>

<template>
  <b-overlay :show="isLoading">
    <!-- <div>{{ student }}</div> -->
    <b-collapse id="collapse-4" v-model="visible" class="mt-2 mb-5">
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
    </b-collapse>
    <div class="row g-3">
      <div class="col-md-2">
        <select @change="getAllStudent" v-model="filterBy" action="" class="form-select">
          <option value="">Filter By</option>
          <option :value="filter.id" v-for="filter in filters" :key="filter.id">
            {{ filter.value }}
          </option>
        </select>
      </div>
      <div class="col-md-2">
        <select @change="getAllStudent" v-model="sortBy" action="" class="form-select">
          <option value="" selected>ASC</option>
          <option value="-">DESC</option>
        </select>
      </div>
      <div class="col-md-3">
        <select
          @change="getAllStudent"
          v-model="batchBy"
          class="form-select"
          name="course"
          id="student-batch"
        >
          <option value="">All Batch</option>
          <option v-for="batch in batches" :key="batch.id" :value="batch.id">
            {{ batch.title }}
          </option>
        </select>
      </div>
      <div class="col-md-2">
        <input
          placeholder="Search..."
          type="input"
          v-model="q"
          class="form-control input-sm"
          @input="getAllStudent"
        />
      </div>
      <div class="col-auto ms-auto">
        <b-button
          variant="primary"
          size="sm"
          :class="visible ? null : 'collapsed'"
          :aria-expanded="visible ? 'true' : 'false'"
          aria-controls="collapse-2"
          @click="visible = !visible"
        >
          Add Student
        </b-button>
      </div>
    </div>

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
                        <a href="#" class="dropdown-item" @click="studentData(i)">Make Payment</a>
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
      ref="paymentHistoryModalRef"
      :student-payment-list="studentPaymentList"
      :student-table-row="studentTableRow"
    />

    <payment-modal
      ref="paymentModalRef"
      v-model:payment-form-data="paymentFormData"
      :student-table-row="studentTableRow"
      @submit-payment-form="submitPaymentForm"
    />
  </b-overlay>
</template>

<style scoped></style>
