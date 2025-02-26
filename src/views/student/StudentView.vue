<script setup>
import { ref, onMounted } from 'vue'
import StudentCreateUpdate from '@/components/student/StudentCreateUpdate.vue'
import PaymentHistoryModal from './PaymentHistoryModal.vue'
import { useTemplateRef } from 'vue'

import axios from 'axios'

const batches = ref([])
const academicYears = ref([])
const homeTowns = ref([])
const colleges = ref([])
const paymentList = ref()

const student = ref({
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
})

const errors = ref({})
const studentsList = ref({ results: [] })
const studentPaymentList = ref([])
const studentTableRow = ref({})

const q = ref('')
const sortBy = ref('')
const batchBy = ref('')
const filterBy = ref('')

const getAllStudent = () => {
  // axios.defaults.xsrfCookieName = 'csrftoken'
  // axios.defaults.xsrfHeaderName = 'X-CSRFTOKEN'
  axios
    .get(
      `http://127.0.0.1:8000/student/filter/?q=${q.value}&filter_by=${sortBy.value}${filterBy.value}&batch=${batchBy.value}`,
    )
    .then((response) => {
      studentsList.value = response.data
      console.log(response.data)
    })
    .catch((error) => {})
}

//get payment history

const getPaymentHistory = (student) => {
  studentTableRow.value = student
  // axios.defaults.xsrfCookieName = "csrftoken";
  // axios.defaults.xsrfHeaderName = "X-CSRFTOKEN";
  axios
    .get(`http://127.0.0.1:8000/course/student-payment-list/${student.id}/`)
    .then((response) => {
      studentPaymentList.value = response.data.results
      openModalRef.value.openModal()
    })
    .catch((error) => {
      console.log(error)
    })
}

onMounted(() => {
  getAllStudent()
})

const counter = ref(0)
const openModalRef = useTemplateRef('openModalRef')
</script>

<template>
  <StudentCreateUpdate v-model:student="student" />
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
</template>

<style scoped></style>
