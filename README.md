如何使用 GLM-4V-Plus API。

首先在 https://bigmodel.cn/usercenter/apikeys 申请一个API Key。

## Install

```bash
pip install -r requirements.txt
```


## Example

```python
import base64

from zhipuai import ZhipuAI

video_path = "test_video.mp4"
# 支持多轮对话
question_list = ["你是谁","请仔细描述一下这个视频"]

stream_output = True
with open(video_path, 'rb') as video_file:
    video_url = base64.b64encode(video_file.read()).decode('utf-8')

api_key="None" # 填写您自己的APIKey
if api_key == "None":
    print("请在代码中填写您的APIKey")
    exit(1)
client = ZhipuAI(api_key=api_key)  

history = []
for question in question_list:
    print("User:", question)
    if history is None or len(history) == 0:
        messages = [
            {
                "role": "user",
                "content": [
                    {
                        "type": "video_url",
                        "video_url": {
                            "url": video_url
                        }
                    },
                    {
                        "type": "text",
                        "text": question
                    }
                ]
            }
        ]
    else:
        messages = [
            {
                "role": "user",
                "content": [
                    {
                        "type": "video_url",
                        "video_url": {
                            "url": video_url
                        }
                    },
                    {
                        "type": "text",
                        "text": history[0][0]
                    }
                ]
            },
            {
                "role": "assistant",
                "content": history[0][1]
            }
        ]
        for (q, a) in history[1:]:
            messages.append(
                {
                    "role": "user",
                    "content": q
                }
            )
            messages.append(
                {
                    "role": "assistant",
                    "content": a
                }
            )
        messages.append(
            {
                "role": "user",
                "content": question
            }
        )

    response = client.chat.completions.create(
        model="glm-4v-plus",
        messages=messages,
        stream=stream_output,
        # temperature=0.1
    )
    if stream_output:
        print("AI:", end="")
        final_text = ""
        for chunk in response:
            current_text = chunk.choices[0].delta.content
            final_text += current_text
            print(current_text, end="")
        print("")
    else:
        final_text = response.choices[0].message.content
        print("AI:", final_text)
    history.append((question, final_text))
```

or 

```shell
python run_example.py
```