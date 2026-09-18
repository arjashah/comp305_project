# Mathematical and Probabilistic Models in Provably Secure Steganography

**Based on:** Nicholas J. Hopper, John Langford, and Luis von Ahn, *Provably Secure Steganography* (2002)

## 1. Introduction

Steganography is the practice of communicating a secret message in such a way that an observer cannot determine that secret communication is taking place. This differs from cryptography, where the main objective is to hide the content of a message. In steganography, the existence of the hidden communication is itself intended to remain undetected.

Hopper, Langford, and von Ahn's *Provably Secure Steganography* develops a formal mathematical foundation for this problem. At the time of the paper, much steganographic research focused on heuristic techniques, such as hiding information in images and video. The authors instead approach steganography from a complexity-theoretic perspective and define security using computational indistinguishability. They also study robustness against an active adversary, referred to as a **warden**, who may modify transmitted information.

The central idea of the paper is to represent ordinary communication as a **probability distribution**. A steganographic encoder samples from this distribution and selects samples that can represent secret information. Probability theory is therefore fundamental to both the construction and analysis of the stegosystem.

This report focuses on the mathematical and probabilistic models used in the paper, particularly the channel distribution, random variables, min-entropy, rejection sampling, computational indistinguishability, and the probabilistic analysis of robustness.

---

## 2. Problem Statement and Communication Model

The basic communication scenario involves **Alice**, **Bob**, and a **warden**. Alice wants to send a hidden message to Bob through a public channel while preventing the warden from detecting that hidden communication is occurring.

The situation can be represented as:

$$
\text{Alice}
\rightarrow
\text{Public Channel}
\rightarrow
\text{Bob}
$$

while the warden observes the communication.

A steganographic system therefore has two main requirements:

1. Bob should be able to recover the hidden message.
2. The warden should not be able to distinguish the resulting stegotext from ordinary channel traffic.

### 2.1 Modelling the Channel

The paper represents a communication channel \(C\) as a distribution over sequences of channel objects. Because future communication can depend on previous communication, the model is conditioned on a **history** \(h\).

For a block of size \(b\), the distribution of the next block is written as

$$
C_h^b.
$$

A random block sampled from this distribution can be represented by

$$
X_h\sim C_h^b.
$$

Here:

* \(C\) = communication channel;
* \(h\) = previous communication history;
* \(b\) = block size;
* \(X_h\) = random channel block.

For example, an email system can be considered a communication channel. The data representing an email forms a channel object, while the complete channel represents a distribution over sequences of emails.

This model is important because the steganographic encoder must produce objects that are consistent with the probability distribution of normal communication.

---

## 3. Probability, Randomness, and Entropy in the Stegosystem

The paper uses several random variables and probability distributions to construct and analyse its steganographic protocols.

The secret key is selected from the uniform distribution:

$$
K\leftarrow U(k),
$$

where \(U(k)\) represents the uniform distribution over all \(k\)-bit strings.

The channel block is represented by

$$
X_h\sim C_h^b.
$$

Unlike the key, the channel block is **not necessarily uniformly distributed**. Its probability is determined by the statistical behaviour of the communication channel.

The paper also uses random functions when defining pseudorandom functions. If \(g\) is selected from

$$
U(L,l),
$$

then \(g\) is a uniformly chosen function mapping \(L\)-bit strings to \(l\)-bit strings.

These distributions allow the authors to compare real cryptographic constructions with ideal random objects.

### 3.1 Minimum Entropy Requirement

For the steganographic constructions to work, the channel must contain sufficient randomness. The paper imposes the condition

$$
H_\infty(C_h^b)>1.
$$

The min-entropy of a random variable \(X\) is

$$
H_\infty(X)
=
-\log_2\left(\max_xP[X=x]\right).
$$

Therefore,

$$
H_\infty(X)>1
$$

implies

$$
\max_xP[X=x]<\frac12.
$$

In simple terms, no individual channel block can occur with probability one-half or greater.

This condition is significant because the encoder relies on repeatedly sampling the channel. If one particular outcome were too common, the channel would contain too little unpredictability to reliably encode hidden information without changing its statistical appearance.

Thus, entropy provides the link between the properties of the cover channel and the probability of successful embedding.

---

## 4. Rejection Sampling and the Steganographic Constructions

The principal technique used by Hopper et al. is **rejection sampling**.

Suppose the encoder needs to represent a hidden bit \(m_i\). It samples a normal channel block \(c\) and applies a function to that block. If the result corresponds to \(m_i\), the block is accepted. Otherwise, it is rejected and another channel sample is taken.

Conceptually:

$$
c\leftarrow C_h^b
$$

$$
\downarrow
$$

$$
F_K(N,c)
$$

$$
\begin{cases}
\text{Match secret value} &\rightarrow \text{Accept }c\\
\text{No match} &\rightarrow \text{Sample again}
\end{cases}
$$

The important property is that the candidate blocks are sampled from the normal channel distribution.

### 4.1 Probability of Successful Sampling

To analyse this process, the paper considers two channel samples:

$$
X_1,X_2\sim C_h^b
$$

and defines

$$
D=\{X_1\neq X_2\}.
$$

The paper derives

$$
P[F(SE,N)=0]
=
\frac12+\frac14P[D].
$$

The minimum-entropy condition ensures that the most probable channel symbol has probability less than \(1/2\). This gives the lower bound

$$
P[D]>\frac12.
$$

Consequently,

$$
P[F(SE,N)=0]>\frac58.
$$

The corresponding failure probability is therefore less than

$$
\frac38.
$$

The significance of this calculation is that the probability of successful embedding can be derived directly from the randomness of the cover channel.

### 4.2 Construction 1: Stateful Steganography

The first construction uses a **pseudorandom function (PRF)**, a secret key \(K\), and a counter \(N\).

For a hidden message bit \(m_i\), the encoder repeatedly samples a channel block \(c\) until

$$
F_K(N,c)=m_i.
$$

The counter makes the construction stateful because the encoding process maintains information about its current position.

The construction can be summarized as:

$$
\text{Message}
\rightarrow
\text{Message bits}
\rightarrow
\text{Channel samples}
\rightarrow
F_K(N,c)
\rightarrow
\text{Accepted blocks}
\rightarrow
\text{Stegotext}.
$$

The security argument relies on the pseudorandomness of \(F_K\). An efficient warden should not be able to distinguish the PRF from a truly random function. Therefore, distinguishing the generated stegotext from ordinary channel traffic would imply an ability to distinguish the underlying pseudorandom function from a random function.

The construction also uses error-correcting codes to reduce the effects of failures in the embedding process.

### 4.3 Construction 2: Stateless Steganography

The second construction avoids maintaining state and instead combines encryption with an **unbiased function**.

Let

$$
f:C\rightarrow\{0,1\}.
$$

The function is unbiased over the channel if

$$
P[f(X)=0]
=
P[f(X)=1]
=
\frac12.
$$

The hidden message is first encrypted:

$$
s=E_K(m).
$$

For each ciphertext portion \(s_i\), the encoder samples channel blocks until

$$
f(c_i)=s_i.
$$

The decoder recovers the ciphertext bits by calculating

$$
s_i=f(c_i)
$$

and then decrypts the result.

The unbiasedness condition makes each required binary value equally likely. However, the paper notes that finding a perfectly unbiased function is a strong requirement and may not be possible for every channel.

The authors therefore also consider approximately unbiased functions. If the function differs from perfect unbiasedness by at most \(\epsilon\), the resulting insecurity can be bounded in terms of \(\epsilon\).

---

## 5. Security and Robustness as Probability Problems

The probability model is also used to define what it means for the stegosystem to be secure.

### 5.1 Computational Indistinguishability

The warden is modelled as a probabilistic adversary. It receives either ordinary channel traffic or stegotext generated by the encoder and attempts to determine which one it has received.

Conceptually, the distinguishing advantage is

$$
Adv(W)
=
\left|
P[W^{SE}=1]
-
P[W^O=1]
\right|.
$$

A secure stegosystem requires this advantage to be **negligible** for efficient adversaries.

A negligible function becomes smaller than \(1/n^c\), for every constant \(c>0\), when the security parameter is sufficiently large.

This gives a mathematical meaning to the statement that stegotext should "look like" ordinary communication. Security is not based on visual similarity or intuition; it is based on the inability of an efficient warden to distinguish two probability distributions.

### 5.2 Robustness Against an Active Warden

The paper also considers an active warden who can modify the transmitted stegotext.

For each relevant position \(i\), define an indicator random variable:

$$
X_i=
\begin{cases}
1,&\text{if the warden modifies position }i,\\
0,&\text{otherwise}.
\end{cases}
$$

The total number of modifications is therefore

$$
\sum_iX_i.
$$

Because the probability of a modification can depend on previous modifications, the authors use conditional expectations and martingale theory. They define quantities of the form

$$
Y_j=
E\left[
\sum_iX_i
\mid
X_1,\ldots,X_j
\right].
$$

The resulting martingale allows the use of **Azuma's inequality** to bound the probability of large deviations from the expected behaviour.

This shows another application of probability theory in the paper: it is used not only to model normal channel behaviour but also to analyse an adversary's modifications.

---

## 6. Discussion and Conclusion

Hopper, Langford, and von Ahn's *Provably Secure Steganography* provides a formal framework for analysing steganography using probability theory and computational complexity.

The communication channel is represented as a conditional probability distribution:

$$
X_h\sim C_h^b.
$$

This allows ordinary communication to be treated as a random process rather than a fixed sequence of data. The minimum-entropy requirement

$$
H_\infty(C_h^b)>1
$$

ensures that the channel contains sufficient unpredictability for the proposed sampling techniques.

Rejection sampling then provides the connection between probability and steganographic embedding. Instead of generating artificial cover data, the encoder repeatedly samples from the genuine channel and accepts a sample when it satisfies the condition needed to represent the secret information.

The two constructions illustrate different ways of applying this principle. Construction 1 uses a pseudorandom function and state information, while Construction 2 combines encryption with an unbiased function. In both cases, the statistical properties of the channel are essential to the security argument.

At the security level, computational indistinguishability measures whether a warden can distinguish stegotext from ordinary channel traffic. For active wardens, indicator variables, martingales, and Azuma's inequality provide additional probabilistic tools for analysing modifications.

The paper ultimately connects these constructions to fundamental cryptographic assumptions. It argues that one-way functions are sufficient for constructing secure steganographic protocols and concludes that, within its asymptotic framework, the existence of secure steganography is equivalent to the existence of one-way functions.

Overall, the mathematical framework can be summarized as:

$$
\boxed{\text{Probability models the channel}}
$$

$$
\boxed{\text{Entropy measures its unpredictability}}
$$

$$
\boxed{\text{Rejection sampling performs the embedding}}
$$

$$
\boxed{\text{Computational indistinguishability defines secrecy}}
$$

Thus, probability is not simply an analytical tool applied after a steganographic system is designed. It is a fundamental part of the model used to define the channel, construct the hidden communication, and prove its security.

## References

Hopper, N. J., Langford, J., & von Ahn, L. (2002). *Provably Secure Steganography*. Carnegie Mellon University, School of Computer Science, Technical Report CMU-CS-02-149.

The paper also discusses earlier work on the Prisoner's Problem, information-theoretic models of steganography, and previous practical approaches to hiding information in digital media.
