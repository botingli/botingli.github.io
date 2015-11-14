---
layout: post
title: Parallel Computing
description: "parallel computing project"
modified: 2015-11-01
tags: [C, Parallel Computing, Code, Class]
image:
  background: darkGray.jpg
---
  I am learning parallel computing this semester, and my project is to find the optimal folution of travelling 
  salesman problem in parallel. Since it is a NP problem, improving the efficiency in algorithm will be extremely hard. In order to increase the performance, we make the computations in parallel. I will demonstrate two ways to implement it in parallel, one with openmp and one with MPI.

# Input: table of 13 cities and traveling costs, seperated by space

  {% highlight c %}
  13 A B C D E F G H I J K L M 
  A 0 3 4 2 9 5 8 17 123 43 9 8 7 
  B 2 0 3 12 15 7 8 9 4 5 4 3 1 
  C 9 5 0 31 8 6 7 5 67 9 10 12 8 
  D 1 9 2 0 81 10 2 18 124 5 5 76 4 
  E 1 2 3 4 0 1 123 12 4 99 46 24 3 
  F 10 9 5 4 3 0 9 99 3 23 44 66 77 
  G 123 2 4 6 8 10 0 9 1 2 22 35 62 
  H 8 78 20 38 40 19 7 0 3 4 14 25 64
  I 1 123 42 56 23 1 2 3 0 5 23 66 65 
  J 235 3 1 577 23 14 6 2 9 0 23 8 5 
  K 1 3 5 6 7 9 2 4 5 7 0 23 9 
  L 5 7 89 25 6 64 13 24 67 3 8 0 12
  M 23 67 25 85 42 85 65 32 79 3 25 11 0

  {% endhighlight %}

## OpenMP version

  {% highlight c %}

#include <stdlib.h>
#include <stdio.h>
#include <iostream>
#include <algorithm>
#include <omp.h>
#define NUM_THREADS 8


int ** Cities;    // the table of cost
char * CityNames; // a list of city names
int * CityNumbers;
int * bestPath;     // a array indicating best path
int lowPrice=0;   // the lowest price possible
int numCities;
int * remains;
int ** eachPerm;  // permutation for each thread


int * Convert(int numToConvert)
// the digits of the array and the nth permutation you want
{
  remains =  (int *)malloc(numCities * sizeof(int));
  int * CNList = (int *)malloc(numCities * sizeof(int));

  for(int i=0; i<numCities; i++)
  {
    int a = CityNumbers[i];
    CNList [i]= a;
  }


  for (int i=0; i<numCities; i++)
  {
    remains[i]=0;
  }

  int numToDiv=1;
  int rem;
  int index=0;
  while(numToConvert > 0)
  //get the array of grab sequence
  {
    rem = numToConvert % numToDiv;
    remains[index]=rem;
    index++;
    numToConvert = int(numToConvert / numToDiv);
    numToDiv++;
  }


  int len=numCities-1;

  int * result= (int *) malloc(numCities * sizeof(int));

  int k=0;
  //grab
  for(int i=len; i >= 0; i--)
  {
    // a = position to get from CNList

    int a= remains[i];
    int get = CNList[a];

    //printf("get: %d", get);

    for (int j=a; j<len; j++)
    {
      //leave no holes
      CNList[j]= CNList[j+1];
    }

    result[k] = get;
    k++;
  }
  return result;
}

int find(int id, int toWork)
{
  int total=0;
  int checked =0;

  for (int q = 0; q < toWork; q++)
  {
    for (int i=0; i<numCities; i++)
    {
      int from = eachPerm[id][i];
      int destination = eachPerm[id][(i+1)%numCities];  //return to the first city when get to the last one
      int trip = Cities[from][destination];
      total += trip;  //calculate the total price
    }

    checked++;
    

    // printf("CPU %d is investigating the No.%d posible route: ", id, checked);
    // for (int i =0; i<numCities; i++)
    // {
    //  printf("%d", eachPerm[id][i]);
    // }
    // printf(", this route cost: %d, current low cost: %d. \n", total, lowPrice);

    if (total<lowPrice)
    {
      //update the lowprice and the sequence if finds a better solution
      #pragma omp critical
      {
        lowPrice=total;

        for(int i=0; i<numCities; i++)
        {
          int a=eachPerm[id][i];
          bestPath[i]=a;
        }
      }
      #pragma omp end critical

    } 

    total= 0; 

    std::next_permutation(eachPerm[id], eachPerm[id]+numCities);
  }
  //while(std::next_permutation(CityNumbers+1, CityNumbers+numCities));// the first city is fixed
}


int main ()
{
  FILE *file = fopen("cityTable.txt", "r");
  char buff[25555];

  fscanf(file, "%s", buff); //get number of cities
  numCities = atoi(buff);
  printf ("total %d cities to go.\n",numCities);

  Cities= (int **) malloc(numCities*sizeof(int *));
  CityNames=(char *) malloc(numCities*sizeof(char));
  CityNumbers=(int *) malloc(numCities*sizeof(int));
  bestPath=(int *) malloc(numCities*sizeof(int));

  for (int i=0; i<numCities; i++)
  {
    bestPath[i]=i; //give bestPath an initial value
  }


  for ( int i=0; i < numCities; i++)
  {
    //get the array of city names
    fscanf(file, "%s", buff);
    CityNames[i]=buff[0];
    printf("%c ", CityNames[i]);
  }

  printf("\n");

  for (int i=0; i<numCities; i++)
  {
    CityNumbers[i]=i;
  }

  for (int i=0; i< numCities; i++)
  {
    fscanf(file, "%s", buff);   //waste a move
    Cities [i]= (int *) malloc(numCities* sizeof(int));
 
    for (int k=0; k<numCities; k++)
    {
      fscanf(file, "%s", buff);
      Cities[i][k]=atoi(buff);    //get the price tables for the cities
    }
  }

  for(int i=0; i<numCities;i++)
  {
    for(int k=0; k<numCities; k++)
    {
      printf("%d ", Cities[i][k]);
    }
    printf("\n");
  }


  for (int i=0; i<numCities; i++)
    {
      int destination=(i+1)%numCities;  
      lowPrice += Cities[i][destination]; //give lowprice an initial value; 
    }


  // calculate number of permutations each thread need to go through
  int numTotalPerm, numEachPerm;
  int hold = 1;
  for (int i = (numCities-1); i>0; i--)
  {
    hold = hold * i;
  }
  numTotalPerm = hold;
  numEachPerm = numTotalPerm/NUM_THREADS;

  printf("TotalPerm: %d, NumEach: %d \n", numTotalPerm, numEachPerm);

  // start to split work for each thread
  int startPos = 0;
  eachPerm = (int **)malloc(NUM_THREADS * sizeof(int *));

  for (int i=0; i<NUM_THREADS; i++)
  {
    int * ptr;
    eachPerm [i] = (int *) malloc (numCities * sizeof (int));

    ptr = Convert(startPos);

    printf("This Thread start: ");
    for( int e =0 ; e<numCities; e++)
    {
      printf("%d", ptr[e]); 
    }
    printf("\n");

    eachPerm[i] = ptr;

    startPos += numEachPerm;
  }

  //printf("Great 3\n");
  //int checked=0;
  int id;
  omp_set_num_threads(NUM_THREADS);


  #pragma omp parallel private(id)
  {
    //printf("Great 4\n");
    id = omp_get_thread_num();
    printf("I'm thread: %d\n", id);

    find(id, numEachPerm);
  }


  //after analysing all possibilities
  printf ("The lowest cost possible is: %d, if using the best route: ", lowPrice);

  for (int i =0; i<numCities; i++)
  {
    printf("-%d", bestPath[i]);
  }
  printf("\n");


  //clean up and exit

  for (int i=0; i<numCities; i++)
  {
    free(Cities[i]);
  }

  free(CityNames);
  free(Cities);
  free(bestPath);
  free(CityNumbers);
  fclose(file);
}

  {% endhighlight %}

## MPI Version
#### the MPI version is slitly different since the threads are all seperate
  I am only posting the different part in *main()*, in which one thread has to gather the results of 
  other threads and find the best route among them.

  The code of *find()* and *Convert()* should have same functionalities.

  {% highlight c %}
//---------------------------------------------------------------------------------
//Parallel part
  int numtasks, rank, len, rc;
  char hostName[MPI_MAX_PROCESSOR_NAME];

  int init;
  MPI_Initialized(&init);

  if (init == 0); //if not been intialized
  {
    rc =  MPI_Init(&argc, &argv);
    if(rc != MPI_SUCCESS)
    {
      printf("Error.\n");
      MPI_Abort(MPI_COMM_WORLD, rc);
    }
  }

  MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  MPI_Get_processor_name(hostName, &len);
  //printf("Rank: %d Running on %s \n", rank, hostName);


// calculate work for each thread
  int startPos = 0;
  startPos += numEachPerm*rank;


  Convert(startPos, CityNumbers);

  printf("Thread %d start: ", rank);
  for( int e =0 ; e<numCities; e++)
  {
    printf("%d", eachPerm[e]);  
  }
  printf("\n");

  
//start to find best route within each thread

  int thisLow = 0;

  thisLow = find(numEachPerm);


//Gather all results and find the best one among them
  MPI_Barrier(MPI_COMM_WORLD);

  int worldLow;
  MPI_Reduce(&thisLow, &worldLow, 1, MPI_INT, MPI_MIN, 1, MPI_COMM_WORLD);


  MPI_Barrier(MPI_COMM_WORLD);
  int gatherLow[numCities];
  MPI_Gather(&thisLow, 1, MPI_INT, gatherLow, 1, MPI_INT, 1, MPI_COMM_WORLD);

  if(rank == 1)
  {
    printf("Gathered Low Costs: ");

    for(int i=0; i<numtasks; i++)
    {
      printf("%d, ", gatherLow[i]);
    }
    printf("\n");
  }
  

  MPI_Barrier(MPI_COMM_WORLD);

  int gatherPath[numtasks][numCities];
  MPI_Gather(bestPath, numCities, MPI_INT, gatherPath, numCities, MPI_INT, 1, MPI_COMM_WORLD);



  if(rank == 1)
  {
    printf("Gathered best Paths: ");

    for(int i=0; i<numtasks; i++)
    {
      for (int k = 0; k<numCities; k++)
      {
        printf("%d, ", gatherPath[i][k]);
      }
      printf("\n");
    }
    
  }



//after analysing all possibilities
  MPI_Barrier(MPI_COMM_WORLD);
  
  if(rank == 1)
  {

    int total=0;
    int checked =0;
    int LP = 99999999;
  
    for (int q = 0; q < numtasks; q++)
    {
      for (int i=0; i<numCities; i++)
      {
        int from = gatherPath[q][i];
        int destination = gatherPath[q][(i+1)%numCities];   //return to the first city when get to the last one

        int cost = Cities[from][destination];
        //printf("from: %d, destination: %d, cost: %d \n", from, destination,cost);
        total += cost;  //calculate the total price
      }
    
      checked++;

      if (total<LP)
      {
        //update the lowprice and the sequence if finds a better solution
        {
          LP=total;

          for(int i = 0; i<numCities; i++)
          {
            int a = gatherPath[q][i];
            finalPath[i] = a;
          }
        }
      }
      total= 0; 
    }
    
    printf ("The lowest cost possible is: %d, if using the best route: ",worldLow);

    for (int i =0; i<numCities; i++)
    {
      printf("-%d", finalPath[i]);
    }
    printf("\n");
  }


  MPI_Finalize();
  {% endhighlight %}