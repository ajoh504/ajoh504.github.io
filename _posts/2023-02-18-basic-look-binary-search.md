---
layout: post
title: A Basic Look at Binary Search
tag: algorithms
excerpt_separator: <!--more-->
---

The binary search algorithm has a straight-forward use-case: take a sorted collection of integers, search for a key, and return the location where the key is stored. <!--more-->If the key is not found in the collection, then return a value such as `null` or `-1`.

There is one major advantage to binary search, and that advantage is speed. Binary search achieves this by continually cutting the collection in half until a match is found. 

With this in mind, let's compare binary search to linear search, which begins at one end of the collection then searches one item at a time until a match is found. For small data sets, a linear search is fine. But for large data sets, there is a lot of wasted time searching linearly through a collection. Binary search solves this problem by applying a faster method for finding a key within the collection.

### A Data Type for Binary Search

To start with, binary search typically requires a sorted array of integers. First, let's define a C# class with a constructor. 

```cs
namespace BinarySearchSample
{
    public class BinarySearch
    {
        private readonly int[] sorted;
        public BinarySearch(int[] sorted)
        {
            this.sorted = sorted;
        }
    }
}
```
Using the above example, any client code can create an object from `BinarySearch` and construct it by passing a sorted array of integers. The `BinarySearch` class will not be responsible for sorting the integers. This task must be handled by the client code.

Now we need a public method that implements the binary search algorithm. We'll call it `Match()`. `Match()` has a parameter `key` in its method signature, and it will search for `key` inside the `sorted` array. The method body will contain an integer `low` that refers to the starting point and `high` that refers to the stopping point.

```cs
public int Match(int key)
{
    int low = 0;
    int high = sorted.Length - 1;
}
```
The next part of the algorithm is where we continually cut the array in half. To achieve this, we'll create a loop that continues as long as `low` is less than or equal to `high`. Then, we'll compare the value of `key` to the integer stored at the middle of the array. If `key` is less than the middle integer, then we reset `high` to equal  the middle index minus 1, thus cutting the array in half. If `key` is greater than the middle integer, then we reset `low` to equal the middle index plus 1, which also cuts the array in half.

```cs
if (key < sorted[mid]) high = mid - 1;
else if (key > sorted[mid]) low = mid + 1;
```
But how do we get the value of `mid`, which refers to the middle index of the sorted array? We could simply divide `high` by 2, but this would create a problem. If we've previously cut the array in half and reset the value of `low`, then `high` divided by 2 does not give us the middle index of the current array. 

So to get mid, first we subtract `low` from `high`. If low is greater than zero, this ensures that cutting high in half truly gives us the middle index.

Lastly, we put the code in a `while` loop. The loop cuts the array in half whenever a match is not found. If the match is found, the method returns the index where the key is stored. If a match is not found by the end of the loop, the method returns `-1`.

Here is the final class:

```cs
namespace BinarySearchSample
{
    public class BinarySearch
    {
        private readonly int[] sorted;
        public BinarySearch(int[] sorted)
        {
            this.sorted = sorted;
        }

        public int Match(int key)
        {
            int low = 0;
            int high = sorted.Length - 1;
            while(low <= high)
            {
                int mid = low + (high - low) / 2;
                if (key < sorted[mid]) high = mid - 1;
                else if (key > sorted[mid]) low = mid + 1;
                else return mid;
            }
            return -1;
        }
    }    
}
```

### Client Code to Test Binary Search

Now we need either a single integer or a set of integers to search for within a `sorted` array. For this example, we'll accept two arguments from the command line. The first argument is the key to search for, and the second argument is a file path. The file path points to a .txt file, which contains a list of integers.

The client code will do the following:
1. Read the integers from the text file into a List
2. Convert the List to a sorted array
3. Create a `BinarySearch` object, then search for the `key` within the sorted array

```cs
namespace BinarySearchSample
{
    public class Program
    {
         // Sample run configuration:
         // $ BinarySearch.exe 45 integers.txt

        public static int Main(string[] args)
        {
            int key;
            string path;
            string errorMessage = """
Error: invalid arguments
Sample run configuration:
$ BinarySearch.exe 45 integers.txt
""";
            // Check args
            if (args.Length < 2)
            {
                Console.WriteLine(errorMessage);
                return 1;
            }

            // Parse args[0]
            if(Int32.TryParse(args[0], out int keyParsed))
            {
                key = keyParsed;
            }
            else 
            {
                Console.WriteLine(errorMessage);
                return 1;
            }

            // Check args[1]
            if(File.Exists(args[1]))
            {
                path = args[1];
            }
            else
            {
                Console.WriteLine("File path does not exist.");
                return 1;
            }

            // Redirect standard input
            using(var reader = new StreamReader(path))
            {
                Console.SetIn(reader); 
                string? line;
                List<int> numbers = new List<int>();

                while ((line = Console.ReadLine()) != null)
                {
                    // Check values
                    if(Int32.TryParse(line, out int valueParsed))
                    {
                        numbers.Add(valueParsed);
                    }
                    else
                    {
                        Console.WriteLine("Cannot parse integer from file.");
                        return 1;
                    }
                    
                }
                
                // Convert to array
                int[] arr = new int[numbers.Count];
                arr = numbers.ToArray();
                Array.Sort(arr);

                BinarySearch binarySearch = new BinarySearch(arr);
                Console.WriteLine(binarySearch.Match(key));
            }
            return 0;
        }
    }
}
```
{% include utterances.html %}
