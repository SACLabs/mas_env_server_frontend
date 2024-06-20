<script>
  import { json } from "@sveltejs/kit";
  import { onMount } from "svelte";
  export let demandButton = "/demand 接口";
  export let pullButton = "/pull接口";
  export let pushButton = "/push 接口";
  export let reportButton = "/report 接口";

  let task_id = "";
  let demand_data;
  let pull_data;
  let push_data;
  let push_status;
  let report_data;
  let show_task_id = false;
  let show_push_data = false;
  let show_pull_data = false;
  let show_report_data = false;
  async function handleDemandUplaod() {
    try {
      // TODO,增加对本地文件读取的功能，此处为demo展示，采用mock数据
      let mockCommand = "test mock command";
      let mockTests = {
        "tests/test_main.py": "def test_add():\n\tassert 1+1 == 2",
      };
      demand_data = JSON.stringify({
        README: mockCommand,
        tests: mockTests,
      });
      const response = await fetch("http://localhost:4000/demand", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: demand_data,
      });
      const data = await response.json();
      task_id = data["uuid"];
      show_task_id = true;
    } catch (error) {
      console.error("请求失败: ", error);
    }
  }

  async function handleMasPush() {
    try {
      let mock_content = {
        result: {
          code_tree:
            "procoder \n \t ├── main.py \n \t └── tests \n \t\t └── test_main.py",
          code_str: {
            "/procoder/main.py":
              "def add(a,b):\n\treturn a+b\nif __name__ == '__main__':\n\tadd(1,1)",
            "/procoder/tests/test_main.py":
              "def test_add():\n\tassert 1+1 == 2",
          },
        },
        history: "test history",
        graph: "test_graph",
      };
      push_data = {
        task_id: task_id,
        content: mock_content,
      };
      const response = await fetch("http://localhost:4000/push", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(push_data),
      });
      const data = await response.json();
      push_status = data["status"];
      show_push_data = true;
    } catch (error) {
      console.error("请求失败: ", error);
    }
  }

  async function handleReportGet() {
    try {
      const response = await fetch(
        `http://localhost:4000/result?task_id=${task_id}`
      );
      report_data = await response.json();
      show_report_data = true;
    } catch (error) {
      console.error("请求失败: ", error);
    }
  }

  async function handleRullGet() {
    try {
      const response = await fetch(`http://localhost:4000/pull`);
      pull_data = await response.json();
      show_pull_data = true;
      console.log(pull_data)
    } catch (error) {
      console.error("请求失败: ", error);
    }
  }
</script>

<div>
  <button on:click={handleRullGet}>{pullButton}</button>
  {#if show_pull_data}
    <pre><code>
            当前env server中demand board中最新的数据 {@html JSON.stringify(
          pull_data,
          null,
          2
        )}
        </code></pre>
  {/if}
</div>

<div>
  <button on:click={handleDemandUplaod}>{demandButton}</button>
  {#if show_task_id}
    <p>
      <span>当前的task id 值是 {task_id}</span>
    </p>

    <pre><code>当前发送的数据是 {demand_data}</code></pre>
  {/if}
</div>

<div>
  <button on:click={handleMasPush}>{pushButton}</button>
  {#if show_push_data}
    <pre><code
        >Mas发送过来的数据是 {@html JSON.stringify(push_data, null, 2)}</code
      ></pre>

    <pre>env server的响应结果为 {push_status}</pre>
  {/if}
</div>

<div>
  <button on:click={handleReportGet}>{reportButton}</button>
  {#if show_report_data}
    <pre><code class="content-text"
        >env server 对mas发送过来的数据处理完成之后，返回的报告是 {@html JSON.stringify(
          report_data,
          null,
          2
        )}</code
      ></pre>
  {/if}
</div>


<style>
  button {
    margin: 0.5rem;
  }

  div {
    width:800px;
    white-space: normal;
    overflow-wrap: break-word;
    word-break: break-all;
  }

  .content-text {
    white-space: pre-wrap;
  }
</style>
