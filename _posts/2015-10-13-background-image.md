---
layout: post
title: ACM Practice problems
description: "Sample post with a background image CSS override."
tags: [Java, Code, ACM]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  background: grey.jpg
---

I am practicing for the ACM contest next month. Some of the problems are real tricky. My best record so far 
is a problem of 8.2 difficulty rating. But even for the comparatively simple problem, there are some elegent 
solutions instead of brute force.

### My sample java solution for the amalgamated problem

The input will be contants for a function. and the last input is number of auto-incremented x values.

The program is deisgned to find the largest deline of values among all points efficiently.

{% highlight java %}
import java.util.*;
import java.lang.Math;

class Amalgamated
{
	static ArrayList<Double> prices;
	public static void main(String[] args)
	{
		Scanner sc = new Scanner(System.in);
		int p, a, b, c, d, n;

		p= sc.nextInt();
		a= sc.nextInt();
		b= sc.nextInt();
		c= sc.nextInt();
		d= sc.nextInt();
		n= sc.nextInt();
		sc.close();
		
		prices = new ArrayList <Double>();

		if((n==0)||(n==1))
		{
			System.out.println(""+0);
			return;
		}

		for (int i=1; i<=n; i++)
		{
			double price= p*(Math.sin(a*i+b)+Math.cos(c*i+d)+2);
			prices.add(price);
		}
		
		System.out.println(findDec(0, prices.size()));
	}

	
	public static double findDec(int start, int end)
	{
		int [] A=findMaxMin(start, end);
		int maxIndex=A[0];
		int minIndex=A[1];
		
		if(maxIndex==minIndex)
		{
			return 0;
		}
		else if (maxIndex<minIndex)
		{
			return (prices.get(maxIndex)-prices.get(minIndex));
		}
		else
		{
			double RMax=prices.get(maxIndex);
			double RMin=prices.get(findMaxMin(maxIndex, prices.size())[1]);
			
			double RMaxDec = RMax-RMin;
			//System.out.println("Rmax:"+RMax+", RMin; "+ RMin+",,RDec: "+RMaxDec);

			double T= Math.max(findDec(0, maxIndex-1), RMaxDec);
			//System.out.println("found");
			return T;
		}
		
	}

	public static int[] findMaxMin(int start, int end)
	{
		double max, min;
		int[] indexs = new int[2];
		max=0;
		min=prices.get(start);

		for(int index=start; index < end; index++)
		{
			double num = prices.get(index);
			
			if (max<num)
			{
				max=num;
				indexs[0]=index;
			}

			if (min>=num)
			{
				min=num;
				indexs[1]=index;
			}
		}
		//System.out.println("Max: "+max+"Min: "+min);
		return indexs;
	}
}
{% endhighlight %}
