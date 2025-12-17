# nitectf writeups

## Warmup

### Description
Welcome to niteCTF 2025. Join our Discord Server: Link

### Solution
The announcements channel in the Discord server had the flag.

### Flag
**Flag:** `nite{h3r3's_t0_4n0th3r_sl33pl3ss_nite}`



## Antakshiri (AI)

### Description
My friend can never remember movie names, absolutely hopeless. This time he only recalls six cast members, nothing else. He's always been dense, but we've been a tight for years, so I guess I'm stuck helping him again. Can you figure out the movie he's trying to remember?

### Solution
So we understand that each of the cast members represents a node that we need to find and we need to sort them in descending order to get the flag. To do this, we used K-Nearest Neighbors (KNN) classification. KNN basically works by first storing all the training data. When a new data point comes in, it looks for the K closest points to it and assigns the class that appears the most among those neighbors.

In our case, we pick one node and use the nearest-neighbors logic to find the 5 closest nodes to it. Whenever we consistently see the same 6 nodes grouping together based on distance, that tells us we’ve found the correct cluster. It’s kind of like how actors from the same movie would always show up together no matter which one you start with.

```
import numpy as np
from sklearn.cluster import KMeans

lv = np.load("latent_vectors.npy") #latent vectors embedding

norms = np.linalg.norm(lv, axis=1) #Euclidian length of vectors
labels = KMeans(n_clusters=2, random_state=0).fit(norms.reshape(-1, 1)).labels_ #Forming clusters

movie_label = 1 if norms[labels == 1].mean() > norms[labels == 0].mean() else 0
movies = np.where(labels == movie_label)[0]
actors = np.where(labels != movie_label)[0]

M = lv @ lv.T #Similarity matrix (gives number of items matrix)

for movie in sorted(movies):
    sims = M[movie, actors] #similarity between the movie and each actor
    order = np.argsort(-sims) #most similar sorting
    top6 = actors[order[:6]] #top 6 actors
    seq = ",".join(str(x) for x in sorted(top6, reverse=True)) #sort the 6 actors in reverse order
    print(f"Movie {movie}: seq={seq}")
```

This prints `Movie 3: seq=189,177,134,108,37,29`, which helps us get the flag.

### Flag:

**Flag:** `nite{Diehard_1891771341083729}`



## floating-point guardian (AI)

### Description
Look at my digital gurdian. I built it using my custom made neural network written in C. Bad move? eh well. Connection: `ncat --ssl floating.chals.nitectf25.live 1337`.

### Solution
The connection leads to a service that asks 15 questions and claims to verify the user using a custom neural network written in C. The service prints a probability called MASTER PROBABILITY and checks whether it matches a hard-coded target value.

From the source code, the key logic is:

```
#define TARGET_PROBABILITY 0.7331337420
#define EPSILON 0.00001


if (fabs(probability - TARGET_PROBABILITY) < EPSILON) {
print_flag();
}
```

So the goal is not to provide realistic human data, but instead to supply any 15 floating‑point numbers such that: `forward_pass(inputs) ≈ 0.7331337420` within a tolerance of `1e-5`. 

We modified the provided code so it reads 15 doubles from stdin and prints only the network output:

```
#include <stdio.h>
#include <math.h>


#define INPUT_SIZE 15


const unsigned char XOR_KEYS[INPUT_SIZE] = {
0x42,0x13,0x37,0x99,0x21,0x88,0x45,0x67,
0x12,0x34,0x56,0x78,0x9A,0xBC,0xDE
};


double W1[15][8] = {
{0.523,-0.891,0.234,0.667,-0.445,0.789,-0.123,0.456},
{-0.334,0.778,-0.556,0.223,0.889,-0.667,0.445,-0.221},
{0.667,-0.234,0.891,-0.445,0.123,0.556,-0.789,0.334},
{-0.778,0.445,-0.223,0.889,-0.556,0.234,0.667,-0.891},
{0.123,-0.667,0.889,-0.334,0.556,-0.778,0.445,0.223},
{-0.891,0.556,-0.445,0.778,-0.223,0.334,-0.667,0.889},
{0.445,-0.123,0.667,-0.889,0.334,-0.556,0.778,-0.234},
{-0.556,0.889,-0.334,0.445,-0.778,0.667,-0.223,0.123},
{0.778,-0.445,0.556,-0.667,0.223,-0.889,0.334,-0.445},
{-0.223,0.667,-0.778,0.334,-0.445,0.556,-0.889,0.778},
{0.889,-0.334,0.445,-0.556,0.667,-0.223,0.123,-0.667},
{-0.445,0.223,-0.889,0.778,-0.334,0.445,-0.556,0.889},
{0.334,-0.778,0.223,-0.445,0.889,-0.667,0.556,-0.123},
{-0.667,0.889,-0.445,0.223,-0.556,0.778,-0.334,0.667},
{0.556,-0.223,0.778,-0.889,0.445,-0.334,0.889,-0.556}
};


double B1[8]={0.1,-0.2,0.3,-0.15,0.25,-0.35,0.18,-0.28};


double W2[8][6]={
{0.712,-0.534,0.823,-0.445,0.667,-0.389},
{-0.623,0.889,-0.456,0.734,-0.567,0.445},
{0.534,-0.712,0.389,-0.823,0.456,-0.667},
{-0.889,0.456,-0.734,0.567,-0.623,0.823},
{0.445,-0.667,0.823,-0.389,0.712,-0.534},
{-0.734,0.623,-0.567,0.889,-0.456,0.389},
{0.667,-0.389,0.534,-0.712,0.623,-0.823},
{-0.456,0.823,-0.667,0.445,-0.889,0.734}
};


double B2[6]={0.05,-0.12,0.18,-0.08,0.22,-0.16};

double W3[6]={0.923,-0.812,0.745,-0.634,0.856,-0.723};
double B3=0.42;


double xor_activate(double x, unsigned char key){
long v=(long)(x*1000000);
v^=key;
return (double)v/1000000.0;
}


double forward(double in[15]){
double h1[8]={0},h2[6]={0},out=0;
for(int j=0;j<8;j++){
for(int i=0;i<15;i++){
double a;
if(i%4==0) a=xor_activate(in[i],XOR_KEYS[i]);
if(i%4==1) a=tanh(in[i]);
if(i%4==2) a=cos(in[i]);
if(i%4==3) a=sinh(in[i]/10.0);
h1[j]+=a*W1[i][j];
}
h1[j]=tanh(h1[j]+B1[j]);
}
for(int j=0;j<6;j++){
for(int i=0;i<8;i++) h2[j]+=h1[i]*W2[i][j];
h2[j]=tanh(h2[j]+B2[j]);
}
for(int i=0;i<6;i++) out+=h2[i]*W3[i];
return 1.0/(1.0+exp(-(out+B3)));
}

int main(){
double x[15]={0};
for(int i=0;i<15;i++) scanf("%lf",&x[i]);
printf("%.10f\n",forward(x));
}
```
We then wrote a Python hill‑climbing solver that randomly tweaks one input at a time and keeps changes only if the error improves:

```
import subprocess
import random


TARGET = 0.733133742
BEST = 999


x = [0.0]*15


for _ in range(200000):
i = random.randint(0,14)
old = x[i]
x[i] += random.uniform(-1,1)


p = subprocess.Popen([
"./solve"],
stdin=subprocess.PIPE,
stdout=subprocess.PIPE
)
out = float(p.communicate(("\n".join(map(str,x)) + "\n").encode())[0])
err = abs(out - TARGET)


if err < BEST:
BEST = err
print("BEST:", BEST)
print(x)


if err > BEST:
x[i] = old


if BEST < 1e-5:
break
```
Running this script converges to an error of approximately 1.1e-6, which satisfies the challenge condition. The solver produced the following 15 values:

```
0.14179660895234547
-0.5740568963279777
-0.05587207631828006
-0.8893991974020987
0
0
0.006565923048292177
0.7129203509769413
-2.0568628205588615
0.8278406741733553
-0.3558478205755702
-0.04969536261004737
-0.6710677494929975
0
0.015899312632646323
```

Using these numbers when prompted for an input gave the following success message:

```
MASTER PROBABILITY: 0.7331348520
You are the master! Here is your flag:
nite{br0_i5_n0t_g0nn4_b3_t4K1n6_any1s_j0bs_34x}
```

### Flag
**Flag:** `nite{br0_i5_n0t_g0nn4_b3_t4K1n6_any1s_j0bs_34x}`



## Bonsai Bankai (OSINT)

### Description
Geoguessr, but make it slightly more ragebait-y.
Submit the correct location within a radius of 250 m to get the flag.

### Solution
We started by looking up the buildings on Google Lens and found a match on a local Japanese real estate site. The building's location was not on the map but the parking lot next to it was listed. This parking lot was 2 blocks down from the station. Using that location, gave us the flag.

### Flag
**Flag:** `nite{0n_th3_g4rden_0f_brok3n_dr3am5}`



## Database Reincursion (WebEx)

### Description
First day as an intern at Citadel Corp and i'm already making strides!
Got rid of that bulky unnecessary security system and implemented my own simple solution.

### Solution (Attempted)
When first interacting with the challenge, the application presented a simple login form that accepted a username and password and consistently returned “Invalid username or password”. Given the challenge context (“intern security”, “simple solution”) and the fact that it was named Database Reincursion, our initial assumption was that this would be a backend vulnerability, most likely SQL injection or authentication logic bypass.

The first approach was to test common SQL injection payloads as well as logic-based inputs like empty credentials or wildcard-style usernames. These attempts consistently failed, and in many cases the application returned: `Citadel SysSec: Input rejected by security filter`.

This indicated the presence of a pre-auth input filter, likely blocking suspicious characters before the request ever reached the backend. Since SQL injection attempts were rejected at input validation level, this ruled out straightforward injection vectors. Browser developer tools revealed that the login form submission was handled via JavaScript rather than a traditional HTML form submission. This raised the possibility that: the filter was client-side and that backend logic might differ from what the frontend enforced.

We found a WebAssembly binary named wasm_feature.wasm was discovered, but we realized that the website never actually loaded the WASM. Returning to the Network tab, closer inspection of the login request revealed tat the request was sent to `POST https://database.chals.nitectf25.live/`
