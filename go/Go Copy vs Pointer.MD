# Go里面使用Copy和使用Pointer对性能的影响

go version go1.14.4 darwin/amd64

### 密集内存分配 Copy vs Pointer



    go test ./... -bench=BenchmarkMemoryStack -benchmem -run=^$ -count=10 > head.txt && benchstat head.txt

#

    name             time/op
    MemoryStack-12   8.60ns ± 7%
    
    name             alloc/op
    MemoryStack-12    0.00B         
    
    name             allocs/op
    MemoryStack-12     0.00     

#

    name            time/op
    MemoryHeap-12   65.2ns ±21%
    
    name            alloc/op
    MemoryHeap-12    96.0B ± 0%    
    
    name            allocs/op
    MemoryHeap-12     1.00 ± 0%

密集内存分配，Copy性能是优于Pointer的，因为Copy主要是在栈上分配，Pointer主要在堆上，需要调用很多GC操作

### 密集函数调用 Copy vs Pointer


    name             time/op
    MemoryStack1-12   305µs ±15%
    
    name             alloc/op   
    MemoryStack1-12   0.00B     
    
    name             allocs/op  
    MemoryStack1-12    0.00 

#

    name            time/op
    MemoryHeap1-12   304µs ±15%
    
    name            alloc/op
    MemoryHeap1-12   0.00B     
    
    name            allocs/op
    MemoryHeap1-12    0.00 

而对于密集函数调用而言，二者几乎没有什么差别。    

