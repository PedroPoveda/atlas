# Pedro Poveda - Static Analysis Project

### Paper title:

**Active Learning of Points-To Specifications** by Osbert Bastani, Alex Aiken, Rahul Sharma, and Percy Liang

## Introduction

Points-to-analysis is a form of static analysis that focuses on which pointers point to which variables or storage location. An example of this is shown below: (Source: Wikipedia)

> For the following example program, a points-to analysis would compute that the points-to set of p is {x, y}.

    int x;
    int y;
    int* p = unknown() ? &x : &y;

Points to analysis has historically been difficult to apply to large libraries. According to the abstract of the paper, a popular solution has been to use points-to-specifications which summarizes only a library's relevant behaviors. This improves precision and handles missing code. According to a different paper by the same lead author Bastani et. al, even though this approach works and is very flexible, for large code bases like the Android framework "many thousands of specifications may be needed", making this impractical. Concretely, there are 30,000 methods in the Android framework and about 10,000 used in an Android app

This paper contributes the open source tool Atlas which automatically infers these points-to-specifications for Android apps. 

## Path Specifications

So what is a path specification? Consider a graph  of an app with all the program semantics.

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled.png)

Now remove the library code which is difficult to analyze. Namely:

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%201.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%201.png)

The result of removing this is below.

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%202.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%202.png)

Those missing links have to be modeled. They are then modeled as 

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%203.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%203.png)

when the library code is unavailable.

## Atlas Summary

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%204.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%204.png)

For a big picture overview of how this algorithm works, consider all the valid specifications that could be written for a block of code. In the above image that is represented by S*. Atlas generates spec S_0 and from there S_1 and so forth as it approximates S*, all the while trying to avoid overgeneralization.

## Results

The authors of the paper had existing handwritten specifications they used as a benchmark for Atlas. Atlas actually inferred 5 times the amount of specifications than the handwritten tools did and got 89% of the handwritten specifications. It did not generate any false positives (meaning specifications that were not accurate), although there were 5 false negatives- meaning it missed some of the handwritten specifications.

It took over a week to write the specifications the authors used in this paper and Atlas is close enough to ground truth to replace the handwritten method. The remaining gap (e.g false negatives) can be filled in by an analyst inspecting the specifications if necessary. This hereby invokes a limitation of Atlas- for applications were soundness is needed, such as compiler optimizations, Atlas alone can not be relied upon.

## Running the Tool

To run the tool it actually was not that difficult. The steps are listed below.

    $ git clone https://github.com/obastani/atlas.git
    $ cd atlas
    $ ant
    $ ant run

Make sure you have ant on your machine - if not you can install it with:

    brew install ant

I ran the Atlas tool on the Java core library.

![Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%205.png](Pedro%20Poveda%20Static%20Analysis%20Project/Untitled%205.png)

Above is a screenshot of the app running. It took about 20 minutes to generate all the specifications for Java default libraries (e.g ArrayList).

## Output

Here is an example of the generated Java specification for ArrayList. Only the first 30 lines are shown for brevity:

    package java.util;
    
    public class ArrayList<E> extends java.util.AbstractList<E> implements java.util.List<E>, java.util.RandomAccess, java.lang.Cloneable, java.io.Serializable {
    	public E f;
    	public ArrayList(java.util.Collection<? extends E> p0) {
    		f = p0.iterator().next();
    	}
    	public ArrayList() {
    	}
    	public boolean add(E p0) {
    		f = p0;
    		return false;
    	}
    	public void add(int p0, E p1) {
    		f = p1;
    	}
    	public boolean remove(java.lang.Object p0) {
    		return false;
    	}
    	public E remove(int p0) {
    		return f;
    	}
    	public E get(int p0) {
    		return f;
    	}
    	public java.lang.Object clone() {
    		ArrayList<E> r = new ArrayList<E>();
    		r.f = f;
    		return r;
    	}
    	public int indexOf(java.lang.Object p0) {
    		return 0;
    	}
    	public void clear() {
    	}
    	public boolean isEmpty() {
    		return false;
    	}
    	public int lastIndexOf(java.lang.Object p0) {
    		return 0;
    	}
    	public boolean contains(java.lang.Object p0) {
    		return false;
    	}
    }

Atlas was run on macOS Mojave (10.14.6). Ant was installed using Homebrew.

## Conclusion

In conclusion, specifications summarizing library code are needed to make points-to-analysis scale. Furthermore, it improves precision and recall as well. The drawback of having to handwrite these specifications is negated when using Atlas, as it automatically can infer them with near soundness without false positives.

## Sources

[https://dl.acm.org/doi/10.1145/3192366.3192383](https://dl.acm.org/doi/10.1145/3192366.3192383) - Paper which this project is based on

[https://www.youtube.com/watch?v=mUy4HKl65WI](https://www.youtube.com/watch?v=mUy4HKl65WI) - Presentation on this paper by the author

[https://drops.dagstuhl.de/opus/volltexte/2019/10803/](https://drops.dagstuhl.de/opus/volltexte/2019/10803/) - Past paper by the same author