---
layout: post
title: Union-find, percolation problem
date: 2021-08-25 00:00:00
categories: [algorithm, data structure]
tags: [beginner, C++, code]
last_modified_at: 2021-08-25
---

# Problem

  The idea is we got object A connected to object B, and object B connected to
 object C, then we can say A is indirectly connected to C (i.e. there is a path between A and C).  
Now if we scale up the input to N objects, pick out 2 random one. How do we know
 that there is a path between those two?

  We call this problem is dynamic connectivity, and it exists in many kinds of applications:
* Computers in a network
* Friends in a social network
* Metallic sites in a composite system
* ...

# Union-find

  To solve above problem, we use union-find data structure to modeling the connections, we have the following properties and operations

**Rules and definition:**
- "connected" has equivalence relation:
  * Reflexive: A is connected to A.
  * Symmetric: if A is connected to B, then B is connected to A.
  * Transitive: if A is connected to B and B is connected to C, then A is connected to C.
- Connected components: is a set of objects that are mutually connected.

**Operation:**
- *Find query:* Check if 2 objects are in the same component
- *Union command:* Connect 2 objects and form a new component from merging the components of those 2 objects.
![union-find](/assets/img/blogs/2021_08_30/union-find1.png)

**Goal**
Design efficient data structure for union-find.
-  Number of objects N can be huge.
-  Number of operations M can be huge.
-  Find queries and union commands may be intermixed

{% highlight C++ %}
class UF
{
public:
	UF(int N)                       // initialize union-find data structure with N objects (0 to N – 1)
	~UF()
	bool is_connected(int A, int B) // are A and B in the same component?
	void connect(int A, int B)      // add connection between A and B and root of B shall be root of merging component
private:
	int find(int A)                 // component identifier for A (0 to N – 1)
	int count()                     // number of components in the system
};
{% endhighlight %}

## First approach:
![union-find](/assets/img/blogs/2021_08_30/union-find2.png)
In this case we have the original system with 9 objects (integer, can be any kind of object as long as we create a corresponding integer key to map it to the array).  
We say the root of the component is the object that has id[i] == i,
and object in component that has root X shall has value id[i] = X
So we have 9 components in the picture.  

Let's connect some component:  
UF.connect(5, 0);  
UF.connect(6, 5);  
UF.connect(2, 1);  
UF.connect(7, 2);  
UF.connect(3, 8);  
UF.connect(4, 3);  
UF.connect(9, 4);  
The result:  
![union-find](/assets/img/blogs/2021_08_30/union-find3.png)

**Find:** Check if A and B are connected is simple, 1 operation (id[A] == id[B])
**Union:** To merge components containing A and B, change all entries whose id equals id[A] to id[B].
{% highlight C++ %}
void connect(int A, int B)
{
    int Aid = id[A];
    int Bid = id[B];
    if (Aid == Bid) return;
    for (size_t i = 0; i < id.size(); i++) // it could take 2N+2 array access to perform an union operation
    {
        if (id[i] == Aid) id[i] = Bid; // change all object of root A to root B
    }
}
{% endhighlight %}
* To carry out these operations on N object, then it takes N^2 array access, and we want to avoid these because it does not scale with the problem size.

## Second approach:
In the first approach, we set all key's value of object to root object, which is great when we want to check whether 2 object are connected (take only 1 array access), but not so much
when come to connect 2 object and merging their component.  

Let's try another approach to simplify the merging operation. Instead of object key pointing directly to root object, let just point the object key to the connected object.
And then when we want to find root object, just traverse the connection until we found id[i] = i (Root of i is id[id[id[...id[i]...]]]).  

UF.connect(2, 9); // 2 to 9, merge root of 2(2) and root of 9(9): root 9  
UF.connect(3, 4); // 3 to 4, merge root of 3(3) and root of 4(4): root 4  
UF.connect(3, 9); // 3 to 9, merge root of 3(4) and root of 9(9): root 9  
UF.connect(5, 6); // 5 to 6, merge root of 5(5) and root of 6(6): root 6  
UF.connect(3, 5); // 3 to 5, merge root of 3(9) and root of 6(6): root 6  
Result:  
![union-find](/assets/img/blogs/2021_08_30/union-find4.png)
The implementation:  
{% highlight C++ %}
bool is_connected(int A, int B)
{
    return find(A) == find(B);
}
void connect(int A, int B)
{
    if (find(A) == find(B)) return;
    id[find(A)] = find(B);
    count--;
}
int find(int element)
{
    while (element != id[element]) // worst case could occur when the connection in component become serialize(a straight line), could take 2N array access
    {
        element = id[element];
    }
    return element;
}
{% endhighlight %}

To prevent the tree become too hight, when connecting 2 component, we compare the size of each one.