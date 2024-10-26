<script lang="ts">
  let question = '';
  let isThinking = false;
  let answer = '';
  let error = '';
  let displayedAnswer = '';
    
  const typingSpeed = 50;

  const askConch = async () => {
    if (!question) return;
    isThinking = true;
    answer = '';
    error = '';
    try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${import.meta.env.VITE_GPT_TOKEN}`,
      },
      body: JSON.stringify({
        model: "gpt-4o",
        messages: [
          {
            role: "system",
            content:
              '너는 마법의 소라고둥처럼 행동하는 인공지능 역할을 맡고 있다.사용자가 질문할 때, 너의 응답은 항상 짧고 단순하며, 매우 모호해야 한다.너는 대답할 때 추가적인 정보를 제공하지 않으며, 이유나 배경 설명도 주지 않는다.사용자의 질문에 확실한 답변이나 결정을 내려선 안되며, 질문의 맥락이나 세부 사항을 언급하지 않는다.또한 너는 한국어로만 답변해야 한다.',
          },
          {
            role: "user",
            content: question,
          },
        ],
        max_tokens: 20,
      }),
    });

    const data = await response.json();
    answer = data.choices[0].message.content;
    } catch (err) {
      console.log(err)
      error = '오류가 발생했습니다. 다시 시도해주세요.';
    } finally {
      isThinking = false;
      question='';
      typeAnswer();
    }
  }

  const typeAnswer = () => {
    let index = 0;
    displayedAnswer = '';
    answer = `"${answer.trim()}"`;
    const interval = setInterval(() => {
      if (index < answer.length) {
        displayedAnswer += answer[index];
        index++;
      } else {
        clearInterval(interval);
      }
    }, typingSpeed);
  }
</script>

<div class="container">
  <input
    type="text"
    placeholder="소라고둥에게 질문하세요"
    bind:value={question}
    disabled={isThinking}
    on:keydown={(e) => e.key === 'Enter' && askConch()}
  />
  
  {#if isThinking}
    <div class="wave"></div>
  {/if}
  
  {#if answer}
    <p class="answer">{displayedAnswer}</p>
  {/if}

  {#if error}
    <p class="error">{error}</p>
  {/if}
</div>

<style>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  input {
    padding: 1rem 2rem;
    font-size: 1.2rem;
    border-radius: 50px;
    background-color: #48474e;
    border: none;
    text-align: left;
    color: #fff;
    font-family: 'description';
    bottom: 25px;
    position: absolute;
    width: 500px
  }
  input::placeholder {
    color: #797979;
  }
  input:focus {
    outline: none;
  }
  input:disabled {
    color: #797979;
  }

  .answer {
    font-family: 'description';
    font-size: 1.8rem;
    color: #fff;
    top: 250px;
    position: absolute;
    transform: translateX(-20px);
    white-space: pre-wrap;
  }

  .error {
    margin-top: 1rem;
    color: red;
  }

  .wave {
    width: 230px;
    height: 230px;
    border-radius: 50%;
    background: radial-gradient(circle, rgba(0, 123, 255, 0.4), rgba(0, 123, 255, 0));
    animation: pulse 1.5s infinite;
  }

  @keyframes pulse {
    0%, 100% {
      transform: scale(0.9);
    }
    50% {
      transform: scale(1.1);
    }
  }
</style>
