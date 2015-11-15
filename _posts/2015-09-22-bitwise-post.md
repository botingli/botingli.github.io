---
layout: post
title: " Do things With Only Bitwise Operations"
description: "Learning to make operations, decision calls with only bitwise operations"
tags: [C, Code, Computer Organization, Class]
image:
  feature: abstract-8.jpg
  credit: dargadgetz
  background: grey.jpg
---

I have learned a lot about how the computer make decisions and discretions with bit-level operations.
Also, although brain-burning, using Bitwise operations in computation intensive-programming can substancially boost performance and efficiency.
So I think these stuff could be useful for my programming contests. 

Here I have collected the labs problems and other puzzles I solved.

## Bit Cowboy Lab -- Integars

{% highlight c %}
int bitAnd(int x, int y) 
{
  /* get  ~(x&y) using ~x|~y */
  
  int c= ~x | ~y;
  return ~c;
}

/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */

int getByte(int x, int n) 
{
  /* shift the word right and leave the disired two digits at the right end, the get using mask */
  int mask=0xff;
  n=n<<3;
  x=x>>(n);
  return x&mask;
}

/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */

int logicalShift(int x, int n) 
{
  /* shift the word arithmatically, then use mask to clear the left-end bytes */
  int mask = (1<<31)>>n; //shift only 31 to prevent overflow
  mask = mask<<1;
  mask=~mask;
  x=x>>n;

  return x&mask;
}


/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) 
{
  /* use mask to check bits , since have operator limits, I count every 4 bits, 
  then shift them rightward step by step, then add them together in the end*/

  int mask1, mask2, mask3, sum;

  mask1 = 0x11 | (0x11 << 8);   //cover 16 bits, 1 for every 4 bits
  mask2 = mask1 | (mask1 << 16);//cover 32 bits, just twice as long as mask1 
  mask3 = 0xF|(0xF<<8);        // mask first 8 bits
  sum =x & mask2;            // count 1's
  sum = sum + ((x >> 1) & mask2);
  sum = sum + ((x >> 2) & mask2);
  sum = sum + ((x >> 3) & mask2);

  //sum calculates the 1's is first 4 bits and other unwanted value
  sum = sum+ (sum>>16);     // get rid of unwanted value

  

  sum = (sum & mask3) + ((sum >> 4) & mask3);
  // limit maximum to 6 bits, because largest decimal value is 32.
  return((sum + (sum >> 8)) & 0x3F);
}


/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x)
{
  /* x or negative x will have 1 as thier first digit, but 0 will not*/
  int mask=0x1;
  int negative = ~x + 1;
  int a = ((x >> 31) & mask);
  int b= ((negative >> 31) & mask);

  // one of a or b will be 1 if x is not 0

  return (a|b)^mask;
}


/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) 
{
  /*juest need to get 0x100000.....*/
  return 1<<31;
}


/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n)
{
  /*if the bits from the left end till n are all same, all 0 or all 1, then the leftside 
  digits can be abadoned and express the same value with only n bits, so the key is to 
  determine wether the first 32 - n bits are all same*/

  int high= 32 + (~n+1);
  int shift= (x<<high)>>high; // this changes all hige order bits to 1 or 0;

  return !(shift^x); // check whether x is still same
}


/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) 
{
  /*needs to round toward 0, so need to know whether x is positive*/

  int mask=(1<<n) + ~0;
    
  int bias= (x>>31)&mask;

    return (x+bias)>>n;
}


/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) 
{
  /* ~x equals  -(2max-abs(x)), thus needs to plus 1*/
  return ~x+1;
}


/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) 
{
  /* if is negative, the first digit is 1, other wise is 0*/

  //then just get the first digit;
  int mask=0x1<<31;
  int firstdigit=x&(mask); // first digit is 0 if x is negative , 1 if x is positive or 0

  return !(firstdigit>>31|!x); //using |!x to count for 0
}


/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) 
{
  /* if x and y has same sign, check the sign of thier differnece, if x and y has different signs
   return 0 if x is positive and if x is negative;*/

  int sign, isLess, dif, equal, isLessorEqual;

  sign = x ^ y;
  isLess = ~sign;
  dif = y + (~x + 1);

  equal = !dif;

  sign = sign & x;
  
  dif = ~dif;
  
  isLess = isLess & dif;
  isLess = isLess | sign;
  isLess = isLess & (1 << 31);

  isLessorEqual = (!!isLess) | (equal);

  return isLessorEqual;  
}


/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) 
{
  /* the value of log2 in binary is actually represented by how many digits are used, the 
    answer shoudld be the number of digits used minus one. I use a method similar to binary search 
    to find the most significant 1*/

    int out=0;

    int a=(!!(x>>16))<<31>>31;// if most sig is in right, a is false, if in left 16 digits a is true;
    out += a & 16;
    x = x>>(a & 16); //shift 16 if is on left side, 


    a = (!!(x>>8)) <<31>>31;   
    out += a & 8;
    x = x>> (a & 8);

    a = (!!(x>>4))<<31>>31;   
    out += a & 4;
    x = x>> (a & 4);

    a = (!!(x>>2))<<31>>31;   
    out += a & 2;
    x = x>> (a & 2);

    a = (!!(x>>1))<<31>>31;   
    out += a & 1;

  return out;
}

{% endhighlight %}

## Bit Cowboy Lab -- Floating numbers

{% highlight c %}

/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) 
{
  /*need to negate the first digit since it represents sign, and keep every thing else;
    use mask to detect special case NaN, and return original
  */

  unsigned exp= uf>>23&0xFF;
  unsigned fra= uf<<9; 
  unsigned a= 0x1<<31;

  if ((exp==0xFF)&&(!!fra))
  {
    return uf;
  }

  return uf^a;
}

/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) 
{
  return 2;
}


/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) 
{
  /* The challenege here is to handle differnet situations , we need to return
  the number for Nan and Infi, add exp by one for normal form and add 1 to the number for 
  denormalized form. I seperate the three parts of the float, make the necessary changes, and 
  combine them to return*/

  unsigned sign = uf>>31 << 31;
  unsigned exp = (((uf << 1) >> 24) << 23);
  unsigned fra= (uf << 9) >> 9;

  if (exp==0x7F800000)
  {
    return uf; //inifinity or NaN
  }
  else if(exp==0)
  {
     return ((uf << 1) | sign); // if in denormal form, shift left by 1 and add sign
  }

  else
  {
    
    exp = (exp + (1 << 23)); //exp + 1 for normal
  }

  return(sign|exp|fra);
{% endhighlight %}




