<script>
    import fastapi from "../lib/api";
    import Error from "../components/Error.svelte"
    import { push } from "svelte-spa-router"
    import moment from 'moment/min/moment-with-locales'
    moment.locale('ko')

    export let params = {}
    let question_id = params.question_id
    //console.log('question_id:' + question_id)
    let question = {answers:[]}
    let content = ""
    let error = {detail:[]}

    function get_question() {
        fastapi("get", "/api/question/detail/" + question_id, {}, (json) => {
            question = json
        })
    }

    get_question()

    function post_answer(event){
        event.preventDefault() // event.preventDefault()는 submit 버튼이 눌릴경우 form이 자동으로 전송되는 것을 방지하기 위해 사용.
        let url = "/api/answer/create/" + question_id
        let params = {
            content: content
        }
        fastapi('post', url, params, (json) => {
            content = '' // 텍스트 박스에 있는 내용이 초기화
            error = {detail:[]}
            get_question() // 새로운 결과값 반영
        },
        (err_json) => {
            error = err_json
        })
    }
</script>

<div class="container my-3">
    <!-- 질문 -->
     <h2 class="border-bottom py-2">{question.subject}</h2>
     <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space; pre-line;">{question.content}</div>
            <div class="d-flex justify-content-end">
                <div class="badge bg-light text-dark p-2">
                    {moment(question.create_date).format("YYYY년 MM월 DD일 a hh:mm")}
                </div>
            </div>
        </div>
     </div> 

     <!-- 이전 질문 목록으로 -->
     <button class="btn btn-secondary" on:click="{() => {
        push('/')
     }}">목록으로</button>

     <!-- 답변 목록 -->
      <h5 class="border-bottom my-3 py-2">{question.answers.length}개의 답변이 있습니다.</h5>
      {#each question.answers as answer}
      <div class="card my-3">
        <div class="card-body">
            <div class="card-text" style="white-space; pre-line;">{answer.content}</div>
            <div class="d-flex justify-content-end">
                <div class="badge bg-light text-dark p-2">
                    {moment(question.create_date).format("YYYY년 MM월 DD일 a hh:mm")}
                </div>
            </div>
        </div>
      </div>
      {/each}
      <!-- 답변 등록 -->
      <Error error={error} />
      <form method="post" class="my-3">
        <div class="mb-3">
            <textarea rows="10" bind:value={content} class="form-control"></textarea>
        </div>
        <input type="submit" value="답변등록" class="btn btn-primary" on:click="{post_answer}" />
      </form>
</div>


<!-- <h1>{question.subject}</h1>
<div>
    작성일시: {question.create_date}
</div>
<div>
    {question.content}
</div>
<ul>
    {#each question.answers as answer}
        <li>{answer.content}</li>
    {/each}
</ul>
<Error error={error} />
<form method="post">
    <textarea rows="15" bind:value={content}></textarea> 
    <input type="submit" value="답변등록" on:click="{post_answer}">
</form>

<style>
    textarea {
        width:100%;
    }
    input[type=submit]{
        margin-top:10px;
    }
</style> -->