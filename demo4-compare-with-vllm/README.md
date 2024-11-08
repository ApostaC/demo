
# Speedup vLLM with LMCache

## introduction

**After this quick 3-minute example, you will know how to install LMCache and demonstrate that it speeds up the inference by 6-17X.**

![image](https://github.com/user-attachments/assets/0da7cb05-58c5-456a-98e2-190b14e03b87)


As shown in the above figure, this demo shows that different vLLM instances can share the prefix KV cache between each other by using LMCache on a single node, so that the KV cache generated by one vLLM instance can be reused by another.

Note that though this demo focuses on single-node case, it can be generalized to allow KV cache sharing between any two vLLM instances in the cluster, as long as they have a commonly-shared NFS disk.

## Prerequisites

- 4 Nvidia A6000 or A40 GPU on the same machine
- Local SSD disk with peak IO bandwidth > 3GB/s (typical speed for SATA3 SSDs)
- docker compose installed on the machine
- sudo access to run docker compose up
- A huggingface token with access to mistralai/Mistral-7B-Instruct-v0.2.
  - _Note: For more information on Huggingface login, please refer to the [Huggingface documentation.](https://huggingface.co/docs/huggingface_hub/en/quick-start)_

Run the demo
```bash
git clone https://github.com/LMCache/demo.git
cd demo/demo4-compare-with-vllm
echo "HF_TOKEN=<your HF token>" >> .env
sudo docker compose up --build -d
timeout 300 bash -c ` until curl -X POST \
    localhost:8000/v1/completions > /dev/null 2>&1; \
    do \
    echo "waiting for server to start..." \
    sleep 1 \
    done` # wait for the docker compose to be ready for receiving requests
```

Please replace <your HF token> with your huggingface token in the bash script above.


## Send your requests to different serving engines



## Steps to follow:

### Send the first request to vLLM w/ LMCache (A)

1. Open `http://<Your server's IP>:8501` (corresponds to the serving engine `vLLM w/ LMCache (A)`)
2. Type query: “What’s FFMPEG (answer in 10 words)”

You should be able to see the following results with a response delay of around 6 seconds.

<img width="333" alt="image" src="https://github.com/user-attachments/assets/e8b388ce-a9d1-44f6-9de2-6a344a52ccfa">



### What if the original vLLM gets a new request with the **same** context

1. Open `http://<Your server's IP>:8502` (corresponds to the serving engine `original vLLM`)
2. Type query: Summarize FFMPEG's main feature in 10 words

You should be able to see the following results with a response delay still of around 6 seconds.

<img width="333" alt="image" src="https://github.com/user-attachments/assets/aff3860f-379f-4130-9df6-7b475e55b90f">


### What if the vLLM w/ LMCache (B) gets a new request with the **same** context

1. Open `http://<Your server's IP>:8503` (corresponds to the serving engine `vLLM w/ LMCache (B)`)
2. Type query: Summarize FFMPEG's main feature in 10 words
  -  You should be able to see the following results with a response delay still of less than 1 second, which is 6x faster than the original vLLM.
3. Type another query: just give me a ffmpeg command line example without any explanation
  - You should be able to see the following results with a response delay still of around 0.35 seconds, which is 15x faster than the original vLLM.

<img width="759" alt="image" src="https://github.com/user-attachments/assets/adc508c5-3487-422b-86bf-5e564356ea7a">

