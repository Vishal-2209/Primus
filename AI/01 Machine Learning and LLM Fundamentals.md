---
tags: [ai, ml, llm, nlp, fundamentals]
created: 2026-08-15
status: reference
---

# Machine Learning and LLM Fundamentals

## Machine Learning ka basic idea

Machine Learning se hum models ko "sochne aur samajhne" ki shakti dete hain, ye baat aasaan bhasha mein bolne ke liye kahi jaati hai. Actual mein aisa literally nahi hota. Neural networks, decision trees, aur jitne bhi types ke models hote hain, wo sab statistical patterns ko data se seekhte hain aur un patterns ke aadhar par predictions ya decisions dete hain. "Sochna" ya "samajhna" jaisa koi literal process andar nahi ho raha hota, ye sirf ek simplified tareeka hai samjhane ka. Iska deep mechanism (neural network ke andar weights, activations, backpropagation, decision tree ke andar splits) abhi ka topic nahi hai, lekin ye samajhna zaroori hai ki jab bhi "model sochta hai" jaisa kuch suna jaaye, uska matlab actually "model pattern match/predict kar raha hai" hota hai.

## LLMs specifically kya karte hain

LLMs (Large Language Models) bhi isi machine learning ke family ka hi part hain, lekin inka specific kaam hai: words (ya technically tokens) ko predict karna. Ek LLM ko jab bhi kuch input diya jaata hai, wo har step par yehi calculate karta hai ki agla token kaunsa aana chahiye, based on us pattern par jo usne training data se seekha hai. Ye process baar baar repeat hota hai jab tak poora response generate na ho jaaye.

Isका matlab ye hai ki LLM ke paas koi literal "understanding" ya "consciousness" nahi hoti, wo bas ek bahut hi powerful next-token predictor hai jiski training itni massive scale par hui hai ki uska output human jaisa coherent aur useful lagta hai.

## NLP ki zaroorat kyun padti hai

LLMs ko humse natural language mein baat karni hoti hai, aur iske liye sirf logic aur patterns kaafi nahi hain, NLP (Natural Language Processing) ki concepts chahiye hoti hain taaki:

- Text ko is tarah represent kiya ja sake ki model use numerically process kar sake
- Language ki structure (grammar, context, meaning) ko kisi hadd tak capture kiya ja sake
- Model ka output wapas readable, coherent language mein convert ho sake

Isi wajah se tokenization jaisi concept important ban jaati hai.

## Tokenization

Tokenization ek process hai jisme raw text ko chhote chhote units, jinhe "tokens" kehte hain, mein todha jaata hai. Ye tokens words ho sakte hain, sub-words ho sakte hain, ya kabhi kabhi single characters bhi. Model text ko directly nahi samajh sakta, wo sirf numbers ke saath kaam karta hai, isliye tokenization ke baad har token ko ek number (token ID) mein convert kiya jaata hai jo model ke andar process hota hai.

```text
Text: "What is the weather?"
Tokens (approx): ["What", " is", " the", " weather", "?"]
Token IDs: [1023, 318, 262, 6193, 30]
```

Isi wajah se LLM APIs mein input aur output ko tokens mein measure kiya jaata hai, aur pricing bhi usi par based hoti hai. Isse related saare metrics (context window, input/output tokens, TPM, etc.) [[LLM Performance and API Terminology]] note mein detail mein cover kiye hain.

## Prompt actually execute kaise hota hai

Hum LLM ko ek prompt de dete hain aur response mil jaata hai, lekin uske peeche jo actually hota hai wo kuch is tarah hai:

```text
User Prompt (text)
        ↓
Tokenization (text → token IDs)
        ↓
Model processes tokens through its layers
        ↓
Model predicts next token, ek ek karke, repeatedly
        ↓
Generated tokens ko wapas text mein convert kiya jaata hai (detokenization)
        ↓
Final Output (text) user ko milta hai
```

Ye process bahut fast hone ke bawajood, deep down ek repetitive next-token-prediction loop hi hai. Yehi wajah hai ki agar prompt ambiguous ya poorly structured ho, to model ka prediction bhi utna hi ambiguous ho sakta hai, kyunki model ke paas sirf pattern-based prediction hi hai, koi real "samajh" nahi hai.

## Ab aage kya

Ye sab groundwork isliye zaroori tha kyunki asli topic ye hai: LLMs ka use karke hum cheezon ko better kaise bana sakte hain. Iske liye hum agents aur workflows design karte hain, jo agle note mein cover kiya hai: [[02 Agents and Workflows]].
