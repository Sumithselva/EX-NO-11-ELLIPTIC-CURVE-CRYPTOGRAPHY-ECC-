# EX-NO-11-ELLIPTIC-CURVE-CRYPTOGRAPHY-ECC

## Aim:
To Implement ELLIPTIC CURVE CRYPTOGRAPHY(ECC)


## ALGORITHM:

1. Elliptic Curve Cryptography (ECC) is a public-key cryptography technique based on the algebraic structure of elliptic curves over finite fields.

2. Initialization:
   - Select an elliptic curve equation \( y^2 = x^3 + ax + b \) with parameters \( a \) and \( b \), along with a large prime \( p \) (defining the finite field).
   - Choose a base point \( G \) on the curve, which will be used for generating public keys.

3. Key Generation:
   - Each party selects a private key \( d \) (a random integer).
   - Calculate the public key as \( Q = d \times G \) (using elliptic curve point multiplication).

4. Encryption and Decryption:
   - Encryption: The sender uses the recipient’s public key and the base point \( G \) to encode the message.
   - Decryption: The recipient uses their private key to decode the message and retrieve the original plaintext.

5. Security: ECC’s security relies on the Elliptic Curve Discrete Logarithm Problem (ECDLP), making it highly secure with shorter key lengths compared to traditional methods like RSA.

## Program:
~~~
#include <stdio.h>
#include <stdlib.h>

/* Elliptic curve over F_p: y^2 = x^3 + a*x + b (mod p) */

typedef struct {
    int x;
    int y;
    int is_infinity; /* boolean: 1 = point at infinity */
} Point;

/* Curve parameters */
const int a = 2;
const int b = 3;
const int p = 17;

/* Normalize value to range [0, m-1] */
int mod_norm(int value, int m) {
    int r = value % m;
    return (r < 0) ? r + m : r;
}

/* Compute modular inverse of k modulo p (returns -1 if none).
   k should be normalized into [0, p-1] first. */
int mod_inverse(int k, int p_mod) {
    k = mod_norm(k, p_mod);
    if (k == 0) return -1;
    for (int x = 1; x < p_mod; x++) {
        if ((k * x) % p_mod == 1) return x;
    }
    return -1;
}

/* Add two points P and Q on the elliptic curve */
Point point_addition(Point P, Point Q) {
    Point R;
    R.is_infinity = 0;

    /* handle identity */
    if (P.is_infinity) return Q;
    if (Q.is_infinity) return P;

    int Px = mod_norm(P.x, p);
    int Py = mod_norm(P.y, p);
    int Qx = mod_norm(Q.x, p);
    int Qy = mod_norm(Q.y, p);

    /* If P == Q, do doubling */
    if (Px == Qx && Py == Qy) {
        if (Py == 0) {               /* tangent is vertical => point at infinity */
            R.is_infinity = 1;
            return R;
        }
        int num = mod_norm(3 * Px * Px + a, p);
        int den = mod_inverse(mod_norm(2 * Py, p), p);
        if (den == -1) {
            R.is_infinity = 1;
            return R;
        }
        int lambda = mod_norm((num * den), p);
        R.x = mod_norm(lambda * lambda - Px - Qx, p);
        R.y = mod_norm(lambda * (Px - R.x) - Py, p);
        return R;
    }

    /* If Px == Qx but Py == -Qy (mod p), then P + Q = O */
    if (Px == Qx && (Py + Qy) % p == 0) {
        R.is_infinity = 1;
        return R;
    }

    /* General addition */
    int num = mod_norm(Qy - Py, p);
    int den_inv = mod_inverse(mod_norm(Qx - Px, p), p);
    if (den_inv == -1) {
        R.is_infinity = 1;
        return R;
    }
    int lambda = mod_norm((num * den_inv), p);
    R.x = mod_norm(lambda * lambda - Px - Qx, p);
    R.y = mod_norm(lambda * (Px - R.x) - Py, p);
    return R;
}

/* Scalar multiplication: compute n * P (double-and-add) */
Point scalar_multiplication(Point P, int n) {
    Point result;
    result.is_infinity = 1;        /* start with point at infinity */
    Point addend = P;

    if (n < 0) {
        /* Not handling negative scalars here; user can implement inverse point if needed */
        fprintf(stderr, "Negative scalar not supported.\n");
        exit(EXIT_FAILURE);
    }

    while (n > 0) {
        if (n & 1) {
            result = point_addition(result, addend);
        }
        addend = point_addition(addend, addend);
        n >>= 1;
    }
    return result;
}

int main(void) {
    Point G = {5, 1, 0}; /* base point */
    int n = 7;

    printf("Base point G: (%d, %d)\n", G.x, G.y);
    Point R = scalar_multiplication(G, n);

    if (R.is_infinity) {
        printf("Result of %d * G: Point at Infinity\n", n);
    } else {
        printf("Result of %d * G: (%d, %d)\n", n, R.x, R.y);
    }
    return 0;
}

~~~


## Output:
<img width="818" height="332" alt="image" src="https://github.com/user-attachments/assets/c5e3cb61-a7e6-493f-b11a-c59ccfd72307" />


## Result:
The program is executed successfully

