# Min-Max Heap in erlang

[Min-max heaps](https://en.wikipedia.org/wiki/Min-max_heap) allow either the largest _or_ the smallest element to be removed, which makes them great for double-ended priority queues.

This implementation stores the heaps as trees of erlang tuples.  It supports heaps where the top level is the smallest (min-max) or largest (max-min), as well as ordinary min-heaps and max-heaps.

Create a min-max heap:

    > H = heap:from_list(mmin,lists:seq(1,10)).
    {heap,mmin,10,{1,{10,{2,{5},{8}},{4,{9},{}}},{7,{3},{6}}}}

Peek at the minimum and maximum elements:

    > heap:min(H).
    {ok,1}
    > heap:max(H).
    {ok,10}

Remove the minimum/maximum elements:

    > {ok,_,H2} = heap:take_min(H).
    {ok,1,{heap,mmin,9,{2,{10,{5,{7},{8}},{4}},{9,{3},{6}}}}}
    > {ok,_,H3} = heap:take_max(H2).
    {ok,10,{heap,mmin,8,{2,{8,{5,{7},{}},{4}},{9,{3},{6}}}}}

Convert the heap back to a list, sorted (I do _not_ recommed using `heap` to sort things; it will never outperform `lists:sort`), but if you're using it as a priority queue, getting a prioritized list out might be helpful during a shutdown sequence:

    > heap:to_list(H3).
    [2,3,4,5,6,7,8,9]

Add an element, and then check the largest/smallest again:

    > H4 = heap:insert(H3,100).
    {heap,mmin,9,{2,{100,{5,{7},{8}},{4}},{9,{3},{6}}}}
    > heap:min(H4).
    {ok,2}
    > heap:max(H4).
    {ok,100}

Create an empty max-min heap:

    > MaxMin0 = heap:new(mmax).
    {heap,mmax,0,{}}

Add some elements in bulk:

    > MaxMin1 = heap:append(MaxMin0,lists:seq(1,10)).
    {heap,mmax,10,{10,{1,{8,{3},{7}},{9,{4},{}}},{2,{5},{6}}}}

You can't get the maximum from a min-heap, nor the minimum from a max-heap:

    > MinHeap = heap:from_list(min,lists:seq(1,10)).
    {heap,min,10,{1,{2,{4,{8},{9}},{5,{10},{}}},{3,{6},{7}}}}
    > heap:max(MinHeap).
    {error,min}
    > MaxHeap = heap:from_list(max,lists:seq(1,10)).
    {heap,max,10,{10,{9,{7,{1},{4}},{8,{3},{}}},{6,{2},{5}}}}
    > heap:min(MaxHeap).
    {error,max}

Trying to take or view and element from an empty heap returns `empty` instead of the `{ok,Value}` or `{ok,Value,NewHeap}`:

    > Empty = heap:new(max).
    {heap,max,0,{}}
    > heap:max(Empty).
    empty
    > heap:take_max(Empty).
    empty
    > heap:min(Empty).
    {error,max}
    > heap:take_min(Empty).
    {error,max}

Note that `{error,_}` is returned in preference to `empty`.  This allows code which is using the wrong operations for the heap type to fail faster.