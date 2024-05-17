<template>
  <div class="h-screen">
    <div class="mx-auto w-3/4 my-6">
      <div class="">
        <table class="table-fixed border border-collapse border-indigo-500 shadow-md rounded-md">
          <thead class=" bg-indigo-500 text-white">
            <tr class="">
              <th class="px-4 py-2 border" colspan="1">Питання</th>
              <th class="px-4 py-2 border" colspan="5">Відповіді</th>
            </tr>
          </thead>
          <tbody>
            <tr class="bg-gray-100 hover:bg-gray-200 hover:text-black w-full">
              <td class="px-4 py-2 border w-1/6">
                <input type="text" placeholder="Питання" v-model="questionInput.QuestionInput">
              </td>
              <td class="px-4 py-2 border w-1/6">
                <div class="answerInput">
                <input class="rounded w-5/6 mr-4" type="text" placeholder="Перша відповідь" v-model="questionInput.answersInput.firstAnswer.firstAnswerTitle">
                <input type="checkbox" v-model="questionInput.answersInput.firstAnswer.firstAnswerValue"  @change="changeState()" v-if="(isRightAnswer===false || questionInput.answersInput.firstAnswer.firstAnswerValue===true)">
              </div>
              </td>
              <td class="px-4 py-2 border">
                <div class="answerInput">
                <input class="rounded w-5/6 mr-4" type="text" placeholder="Друга відповідь" v-model="questionInput.answersInput.secondAnswer.secondAnswerTitle">
                <input type="checkbox" v-model="questionInput.answersInput.secondAnswer.secondAnswerValue"  @change="changeState()" v-if="(isRightAnswer===false || questionInput.answersInput.secondAnswer.secondAnswerValue===true)">
              </div>
              </td>
              <td class="px-4 py-2 border">
                <div class="answerInput">
                <input class="rounded w-5/6 mr-4" type="text" placeholder="Третя відповідь" v-model="questionInput.answersInput.thirdAnswer.thirdAnswerTitle">
                <input type="checkbox" v-model="questionInput.answersInput.thirdAnswer.thirdAnswerValue" @change="changeState()" v-if="(isRightAnswer===false || questionInput.answersInput.thirdAnswer.thirdAnswerValue===true)">
              </div>
              </td>
              <td class="px-4 py-2 border">
                <div class="answerInput">
                <input class="rounded w-5/6 mr-4" type="text" placeholder="Четверта відповідь" v-model="questionInput.answersInput.fourAnswer.fourAnswerTitle">
                <input type="checkbox" v-model="questionInput.answersInput.fourAnswer.fourAnswerValue"  @change="changeState()" v-if="(isRightAnswer===false || questionInput.answersInput.fourAnswer.fourAnswerValue===true)">
              </div>
              </td>
              <td class="px-4 py-2 border">
                <div class="flex justify-around">
                  <button @click="addQuestion()">
                    Додати
                  </button>
                  <button @click="clearQuestion()">
                    Очистити
                  </button>
                </div>
              </td>
            </tr>
            <tr v-for="question in questions" :key="question._id" class="bg-gray-100 hover:bg-gray-200 hover:text-black w-full">
              <td v-if="question._id!=editingQuestion._id" class="px-4 py-2 border w-1/6 text-wrap break-words">{{ question.Question }}</td>
              <td v-else>
                  <input type="text" v-model="editingQuestion.Question">
                </td>
              <td v-for="(index, key, value) in question.answers" class="px-4 py-2 border w-1/6">
                <td  @click="test(index, key, value)" class="px-4 py-2">{{ key }}</td>
                <td v-if="question._id!=editingQuestion._id" class="px-4 py-2"><input type="checkbox" _id="checkbox" v-model="question.answers[key]" disabled></td>
                <td v-else-if="isRightAnswerChange===false || editingQuestion.answers[key]===true" @change="changeStateEdit(editingQuestion.answers)" class="px-4 py-2"><input type="checkbox" _id="checkbox" v-model="question.answers[key]"></td>
              </td>
              <td class="px-4 py-2 border" v-if="question._id!=editingQuestion._id && isEdit==false">
                <div class="flex justify-between">
                  <button @click="editQuestion(question)">Редагувати</button>
                  <button  @click="deleteQuestion(question._id)">Видалити</button>
                </div>
              </td>
              <td class="px-4 py-2 border" v-else-if="question._id==editingQuestion._id && isEdit==true">
                <div class="flex justify-between">
                  <button @click="confirmEdit()">Підтвердити</button>
                  <button @click="endEdit()">Відмовити</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
        <!-- <p v-bind="questionInput">{{ questionInput.answersInput.secondAnswer.secondAnswerValue }}</p> -->
      </div>
    </div>
</div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const questions = ref([]);
const question = ref({Question: "", answers: {}});
const questionInput = ref({QuestionInput: "", answersInput: {
  firstAnswer: {
    firstAnswerTitle: "", 
    firstAnswerValue: false
  }, 
  secondAnswer: {
    secondAnswerTitle: "", 
    secondAnswerValue: false
  }, 
  thirdAnswer: {
    thirdAnswerTitle: "", 
    thirdAnswerValue: false
  }, 
  fourAnswer: {
    fourAnswerTitle: "", 
    fourAnswerValue: false
  }}})
const isRightAnswer = ref(false)
const isRightAnswerChange =ref(true)
const isEdit = ref(false);
const editingQuestion = ref({_id: "", Question: "", answers: {}});
const TEST = ref({"asdasd": {"1":false, "2":true, "3":false, "4":false}, "slovoa": "somequestion", })

function test(index, key, value) {
  console.log(index, key, value)
}
function editQuestion(question) {
  this.isEdit = true;
  this.editingQuestion = question
  console.log(editingQuestion.value)
}
function confirmEdit() {
  this.updateQuestions();
  this.editingQuestion = {_id: "", Question: "", answers: {}}
  this.isEdit=false;
}
function endEdit() {
  this.isEdit = false;
  this.editingQuestion = {_id: "", Question: "", answers: {}}
}
function addQuestion() {
  if(this.questionInput.QuestionInput !='' && 
  this.questionInput.answersInput.firstAnswer.firstAnswerTitle != '' && 
  this.questionInput.answersInput.secondAnswer.secondAnswerTitle != '' && 
  this.questionInput.answersInput.thirdAnswer.thirdAnswerTitle != '' && 
  this.questionInput.answersInput.fourAnswer.fourAnswerTitle != '' && 
  (this.questionInput.answersInput.firstAnswer.firstAnswerValue === true || 
  this.questionInput.answersInput.secondAnswer.secondAnswerValue === true || 
  this.questionInput.answersInput.thirdAnswer.thirdAnswerValue === true || 
  this.questionInput.answersInput.fourAnswer.fourAnswerValue === true)) {

    this.question = {Question: this.questionInput.QuestionInput, answers:{}}
    this.question.answers[this.questionInput.answersInput.firstAnswer.firstAnswerTitle] = this.questionInput.answersInput.firstAnswer.firstAnswerValue
    this.question.answers[this.questionInput.answersInput.secondAnswer.secondAnswerTitle] = this.questionInput.answersInput.secondAnswer.secondAnswerValue
    this.question.answers[this.questionInput.answersInput.thirdAnswer.thirdAnswerTitle] = this.questionInput.answersInput.thirdAnswer.thirdAnswerValue
    this.question.answers[this.questionInput.answersInput.fourAnswer.fourAnswerTitle] = this.questionInput.answersInput.fourAnswer.fourAnswerValue
    this.postQuestion()
  }
  // console.log(this.questionInput.QuestionInput !='' && 
  // this.questionInput.answersInput.firstAnswer.firstAnswerTitle != '' && 
  // this.questionInput.answersInput.secondAnswer.secondAnswerTitle != '' && 
  // this.questionInput.answersInput.thirdAnswer.thirdAnswerTitle != '' && 
  // this.questionInput.answersInput.fourAnswer.fourAnswerTitle != '' && 
  // (this.questionInput.answersInput.firstAnswer.firstAnswerValue === true || 
  // this.questionInput.answersInput.secondAnswer.secondAnswerValue === true || 
  // this.questionInput.answersInput.thirdAnswer.thirdAnswerValue === true || 
  // this.questionInput.answersInput.fourAnswer.fourAnswerValue === true))
  // console.log(this.questionInput.QuestionInput)
}
function clearQuestion() {
  this.questionInput.QuestionInput = ''
  this.questionInput.answersInput.firstAnswer.firstAnswerTitle = '' 
  this.questionInput.answersInput.secondAnswer.secondAnswerTitle = ''
  this.questionInput.answersInput.thirdAnswer.thirdAnswerTitle = ''
  this.questionInput.answersInput.fourAnswer.fourAnswerTitle = ''
  this.questionInput.answersInput.firstAnswer.firstAnswerValue = false
  this.questionInput.answersInput.secondAnswer.secondAnswerValue = false
  this.questionInput.answersInput.thirdAnswer.thirdAnswerValue = false
  this.questionInput.answersInput.fourAnswer.fourAnswerValue = false
  this.isRightAnswer = false
}
function postQuestion() {
  axios.post("http://localhost:3000/questions/", this.question)
    .then(response => {
      // console.log(response.data);
      this.getQuestion();
      this.student = {Question: "", answers: {}}
})}
function changeState() {
  console.log("before", this.isRightAnswer)
  if(this.isRightAnswer==true) {
    this.isRightAnswer = false
  } else if(this.isRightAnswer==false) {
    this.isRightAnswer = true
  }
  
  console.log("after", this.isRightAnswer)
  console.log(this.questionInput.answersInput.secondAnswer.secondAnswerValue ,this.questionInput.answersInput.secondAnswer.secondAnswerValue==true)
}
function changeStateEdit(obj) {
  if(this.isRightAnswerChange==true) {
    this.isRightAnswerChange = false
    console.log("isRightAnswerChange = false")
  } else if(this.isRightAnswerChange==false) {
    this.isRightAnswerChange = true
    console.log("isRightAnswerChange = true")
  }
  console.log(obj)
}
function updateQuestions() {
  axios.put(`http://localhost:3000/questions/${this.editingQuestion._id}`, this.editingQuestion)
  .then((data) =>{
    console.log(data);
    this.getQuestion();
  })
  .catch((error) => {
      console.error(error)
  })
}
function deleteQuestion(questionId) {
  axios.delete(`http://localhost:3000/questions/${questionId}`)
  .then(data => {
    console.log(data);
    this.getQuestion();
  })
}
const getQuestion = async () => {
  try {
    const response = await axios.get('http://localhost:3000/questions');
    questions.value = response.data;
    console.log(response.data)
  } catch (error) {
    console.error('Error fetching questions:', error);
  }
};

onMounted(() => {
  getQuestion();
});
</script>
<style>
  html, body {
    margin: 0;
    padding: 0;
    background-color: #CBD5E0
  }
</style>