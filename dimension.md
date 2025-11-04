## **📊 COMPLETE DIMENSION TRACKING TABLE**

| **Stage** | **Input Shape** | **Operation** | **Output Shape** | **Key Parameters** |
|-----------|----------------|---------------|------------------|-------------------|
| **ENCODER** |
| Raw Input | Text | Tokenization | [18] tokens | - |
| Add Batch | [18] | unsqueeze(0) | [1, 18] | - |
| Embedding | [1, 18] | nn.Embedding | [1, 18, 512] | vocab_size=16465, d_model=512 |
| Pos Encoding | [1, 18, 512] | sin/cos | [1, 18, 512] | - |
| Enc Self-Attn Q | [1, 18, 512] | Linear | [1, 18, 512] | - |
| Split Heads Q | [1, 18, 512] | reshape | [1, 8, 18, 64] | n_heads=8 |
| Attention Scores | [1,8,18,64] × [1,8,64,18] | matmul | [1, 8, 18, 18] | - |
| Scale | [1, 8, 18, 18] | / sqrt(64) | [1, 8, 18, 18] | sqrt(d_k)=8 |
| Softmax | [1, 8, 18, 18] | softmax | [1, 8, 18, 18] | dim=-1 |
| Apply to V | [1,8,18,18] × [1,8,18,64] | matmul | [1, 8, 18, 64] | - |
| Concat Heads | [1, 8, 18, 64] | reshape | [1, 18, 512] | - |
| FFN Layer 1 | [1, 18, 512] | Linear+ReLU | [1, 18, 2048] | d_ff=2048 |
| FFN Layer 2 | [1, 18, 2048] | Linear | [1, 18, 512] | - |
| **Encoder Output** | - | - | **[1, 18, 512]** | Goes to decoder |
| **DECODER (Iteration t)** |
| Target Input | [1, t] | - | [1, t] | t = current length |
| Embedding | [1, t] | nn.Embedding | [1, t, 512] | Same embedding as encoder |
| Pos Encoding | [1, t, 512] | sin/cos | [1, t, 512] | - |
| Dec Self-Attn | [1, t, 512] | Multi-Head | [1, t, 512] | With causal mask |
| Cross-Attn Q | [1, t, 512] | Linear | [1, 8, t, 64] | From decoder |
| Cross-Attn K | [1, 18, 512] | Linear | [1, 8, 18, 64] | From encoder |
| Cross-Attn V | [1, 18, 512] | Linear | [1, 8, 18, 64] | From encoder |
| Cross Scores | [1,8,t,64] × [1,8,64,18] | matmul | [1, 8, t, 18] | Decoder→Encoder attn |
| Cross Apply | [1,8,t,18] × [1,8,18,64] | matmul | [1, 8, t, 64] | - |
| FFN | [1, t, 512] | 512→2048→512 | [1, t, 512] | - |
| **Decoder Output** | - | - | **[1, t, 512]** | - |
| **OUTPUT** |
| Projection | [1, t, 512] | Linear | [1, t, 16465] | vocab_size |
| Last Position | [1, t, 16465] | slice [:,-1,:] | [1, 16465] | Logits for next token |
| Softmax | [1, 16465] | softmax | [1, 16465] | Probabilities |
| Argmax | [1, 16465] | argmax | [1] | Next token ID |
