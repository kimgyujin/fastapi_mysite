<script>
    import fastapi from "../lib/api"
    import { link } from 'svelte-spa-router'
    // 페이지를 유지하기 위해 스토어 변수 사용
    import { page } from "../lib/store"
    import moment from 'moment/min/moment-with-locales'
    moment.locale('ko')

    let question_list = []
    let size = 10
    //let page = 0
    let total = 0
    $: total_page = Math.ceil(total/size)

    function get_question_list(_page) {
        let params = {
            page: _page,
            size: size,
        }

        fastapi('get', '/api/question/list', params, (json) => {
            question_list = json.question_list
            $page = _page
            total = json.total
        })
    }

    $: get_question_list($page)
</script>

<div class="container my-3">
    <table class="table">
        <thead>
            <tr class="table-dark">
                <th>번호</th>
                <th>제목</th>
                <th>작성일시</th>
            </tr>
        </thead>
        <tbody>
            {#each question_list as question, i}
            <tr>
                <td>{ total - ($page * size) - i}</td>
                <td>
                    <a use:link href="/detail/{question.id}">{question.subject}</a>
                    {#if question.answers.length > 0}
                    <span class="text-danger small mx-2">{question.answers.length}</span>
                    {/if}
                </td>
                <td>{moment(question.create_date).format("YYYY년 MM월 DD일 a hh:mm")}</td>
            </tr>
            {/each}
        </tbody>
    </table>
    <!-- 페이징처리 시작 -->
    <ul class="pagination justify-content-center">
        <!-- 이전페이지 -->
        <li class="page-item {$page<=0 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list($page-1)}">이전</button>
        </li>
        <!-- 페이지번호 -->
        {#if $page-5 >= 1}
        <li class="page-item">
            <button class="page-link" on:click="{() => get_question_list($page-5)}">...</button>
        </li>
        {/if}
        {#each Array(total_page) as _, loop_page}
        {#if loop_page >= $page-5 && loop_page <= $page+5}
        <li class="page-item {loop_page === $page && 'active'}">
            <button on:click="{() => get_question_list(loop_page)}" class="page-link">{loop_page+1}</button>
        </li>
        {/if}
        {/each}
        {#if $page+5 <= total_page-2}
        <li class="page-item">
            <button class="page-link" on:click="{() => get_question_list($page+5)}">...</button>
        </li>
        {/if}        
        <!-- 다음 페이지 -->
        <li class="page-item {$page >= total_page-1 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list($page+1)}">다음</button>
        </li>         
    </ul>
    <ul class="pagination justify-content-center">
        <!-- 처음페이지 -->
        <li class="page-item {$page<=0 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list(0)}">처음</button>
        </li>        
        <!-- 마지막 페이지 -->
        <li class="page-item {$page >= total_page-1 && 'disabled'}">
            <button class="page-link" on:click="{() => get_question_list(total_page-1)}">마지막</button>
        </li>  
    </ul>
    <!-- 페이징처리 끝 -->
    <a use:link href="/question-create" class="btn btn-primary">질문 등록하기</a>
</div>

<!-- <ul>
    {#each question_list as question}
        <li><a use:link href="/detail/{question.id}">{question.subject}</a></li>
    {/each}
</ul> -->