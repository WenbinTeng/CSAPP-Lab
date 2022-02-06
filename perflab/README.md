# CS:APP Performance Lab

### 1. Rotate

Matrix rotation can be described as

<img src="https://latex.codecogs.com/svg.image?A(i,j)=A(j,i),&space;\space&space;0&space;\leq&space;i,j&space;\leq&space;n&space;" title="A(i,j)=A(j,i), \space 0 \leq i,j \leq n " />

, which naive baseline is shown below.

```c
#define RIDX(i,j,n) ((i)*(n)+(j))

void naive_rotate(int dim, pixel *src, pixel *dst)
{
    int i, j;
    for (i = 0; i < dim; i++)
        for (j = 0; j < dim; j++)
            dst[RIDX(dim - 1 - j, i, dim)] = src[RIDX(i, j, dim)];
}
```

```
Rotate: Version = naive_rotate: Naive baseline implementation:
Dim		64	128	256	512	1024	Mean
Your CPEs	1.4	1.5	3.7	5.5	7.7
Baseline CPEs	14.7	40.1	46.4	65.9	94.5
Speedup		10.3	26.3	12.7	12.0	12.2	13.8
```

##### Reusing Computation Result

For every inner iteration,
<img src="https://latex.codecogs.com/svg.image?i&space;\times&space;n" title="i \times n" />
will be computed repeatedly, which causes performance loss. We can put the computation of
<img src="https://latex.codecogs.com/svg.image?i&space;\times&space;n" title="i \times n" />
outside of the inner loop.

```c
void rotate(int dim, pixel *src, pixel *dst) 
{
    int i, j;
    int s, t;

    s =  dim * (dim - 1);
    for (i = 0; i < dim; i++)
    {
        t = i * dim;
        for (j = 0; j < dim; j++)
        {
            dst[s - j * dim + i] = src[t + j];
        }
    }
}
```

```
Rotate: Version = rotate: Current working version:
Dim		64	128	256	512	1024	Mean
Your CPEs	1.3	1.5	3.6	5.3	7.6
Baseline CPEs	14.7	40.1	46.4	65.9	94.5
Speedup		11.5	27.3	13.0	12.5	12.4	14.4
```

##### Cache Accessing Optimization

For a large dimension matrix, the memory access of `dst` and `src` will take long strides. We can unroll the inner loop to fit the size of cache. For computation convenience, we can simply unroll the parentheses like

<img src="https://latex.codecogs.com/svg.image?dst((n-1-j)&space;\times&space;n&space;&plus;&space;i)&space;\iff&space;dst((n&space;\times&space;n&space;-&space;n)&space;&plus;&space;i&space;-&space;j&space;\times&space;n)" title="dst((n-1-j) \times n + i) \iff dst((n \times n - n) + i - j \times n)" />

.We set the base address `n^2-n` to `dst` and `0` to `src`, and decrease `n` for `dst` and increase `1` for `src` in a inner loop iteration.

```c
void rotate(int dim, pixel *src, pixel *dst)
{
    int BLOCK = 32; // Images' dimension are multiples of 32

    dst += (dim * dim - dim);

    for (int i = 0; i < dim / BLOCK; ++i)
    {
        for (int j = 0; j < dim; ++j)
        {
            for (int k = 0; k < BLOCK; ++k)
            {
                dst[k] = src[k * dim];
            }

            src += 1;
            dst -= dim;
        }

        src += BLOCK * dim - dim;
        dst += BLOCK + dim * dim;
    }
}
```

```
Rotate: Version = rotate: Current working version:
Dim		64	128	256	512	1024	Mean
Your CPEs	1.1	1.1	1.1	1.5	2.0
Baseline CPEs	14.7	40.1	46.4	65.9	94.5
Speedup		13.2	35.9	43.6	44.3	46.8	33.6
```

### 2. Smooth

The smooth value of an element in matrix is defined as 

<img src="https://latex.codecogs.com/svg.image?Smooth(A(i,j))&space;=&space;\frac{1}{\sum&space;e(i,j)}&space;\sum_{s=-1}^1&space;\sum_{t=-1}^1&space;A(i&plus;s,j&plus;t),&space;\&space;subject&space;\&space;to\begin{cases}A(i,j)=0,&space;\&space;if&space;\&space;i<1&space;\&space;or&space;\&space;i>n&space;\&space;or&space;\&space;j<1&space;\&space;or&space;\&space;j>n&space;\\e(i,j)=1,&space;\&space;if&space;\&space;1&space;\leq&space;i,j&space;\leq&space;n\end{cases}" title="Smooth(A(i,j)) = \frac{1}{\sum e(i,j)} \sum_{s=-1}^1 \sum_{t=-1}^1 A(i+s,j+t), \ subject \ to\begin{cases}A(i,j)=0, \ if \ i<1 \ or \ i>n \ or \ j<1 \ or \ j>n \\e(i,j)=1, \ if \ 1 \leq i,j \leq n\end{cases}" />

, which naive baseline is shown below.

```c
typedef struct {
	unsigned short red;
    unsigned short green;
    unsigned short blue;
} pixel;

typedef struct {
    int red;
    int green;
    int blue;
    int num;
} pixel_sum;

static void initialize_pixel_sum(pixel_sum *sum) 
{
    sum->red = sum->green = sum->blue = 0;
    sum->num = 0;
    return;
}

static void accumulate_sum(pixel_sum *sum, pixel p) 
{
    sum->red += (int) p.red;
    sum->green += (int) p.green;
    sum->blue += (int) p.blue;
    sum->num++;
    return;
}

static void assign_sum_to_pixel(pixel *current_pixel, pixel_sum sum) 
{
    current_pixel->red = (unsigned short) (sum.red/sum.num);
    current_pixel->green = (unsigned short) (sum.green/sum.num);
    current_pixel->blue = (unsigned short) (sum.blue/sum.num);
    return;
}

static pixel avg(int dim, int i, int j, pixel *src) 
{
    int ii, jj;
    pixel_sum sum;
    pixel current_pixel;

    initialize_pixel_sum(&sum);
    for (ii = max(i - 1, 0); ii <= min(i + 1, dim - 1); ii++)
        for (jj = max(j - 1, 0); jj <= min(j + 1, dim - 1); jj++)
            accumulate_sum(&sum, src[RIDX(ii, jj, dim)]);

    assign_sum_to_pixel(&current_pixel, sum);
    return current_pixel;
}

void naive_smooth(int dim, pixel *src, pixel *dst)
{
    int i, j;
    for (i = 0; i < dim; i++)
        for (j = 0; j < dim; j++)
            dst[RIDX(i, j, dim)] = avg(dim, i, j, src);
}
```

```
Smooth: Version = naive_smooth: Naive baseline implementation:
Dim		32	64	128	256	512	Mean
Your CPEs	44.1	44.4	41.4	46.1	46.7
Baseline CPEs	695.0	698.0	702.0	717.0	722.0
Speedup		15.7	15.7	17.0	15.6	15.4	15.9
```

According to the definition of Smooth, we can classify pixels into three categories.

<img src="https://latex.codecogs.com/svg.image?pixels\begin{cases}center&space;\&space;pixels&space;\\others&space;\begin{cases}&space;edges&space;&space;\begin{cases}&space;&space;top&space;\\&space;&space;down&space;\\&space;&space;left&space;\\&space;&space;right&space;&space;\end{cases}&space;&space;\\&space;corners&space;&space;\begin{cases}&space;&space;upper&space;\&space;left&space;\\&space;&space;upper&space;\&space;right&space;\\&space;&space;lower&space;\&space;left&space;\\&space;&space;lower&space;\&space;right&space;\\&space;&space;\end{cases}&space;\end{cases}\end{cases}" title="pixels\begin{cases}center \ pixels \\others \begin{cases} edges \begin{cases} top \\ down \\ left \\ right \end{cases} \\ corners \begin{cases} upper \ left \\ upper \ right \\ lower \ left \\ lower \ right \\ \end{cases} \end{cases}\end{cases}" />

Then we can avoid using `min` and `max` in every loop iteration, which will bring the consumption of branch prediction. The code after unrolling is shown below.

```c
void smooth(int dim, pixel *src, pixel *dst)
{
    int i, j;
    int ni, ci, pi;
    pixel_sum sum;
    pixel current_pixel;

    // top-left
    i = 0;
    j = 0;
    initialize_pixel_sum(&sum);
    accumulate_sum(&sum, src[RIDX(i, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i, j + 1, dim)]);
    accumulate_sum(&sum, src[RIDX(i + 1, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i + 1, j + 1, dim)]);
    assign_sum_to_pixel(&current_pixel, sum);
    dst[RIDX(i, j, dim)] = current_pixel;

    // top-right
    i = 0;
    j = dim - 1;
    initialize_pixel_sum(&sum);
    accumulate_sum(&sum, src[RIDX(i, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i, j - 1, dim)]);
    accumulate_sum(&sum, src[RIDX(i + 1, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i + 1, j - 1, dim)]);
    assign_sum_to_pixel(&current_pixel, sum);
    dst[RIDX(i, j, dim)] = current_pixel;

    // bottom -left
    i = dim - 1;
    j = 0;
    initialize_pixel_sum(&sum);
    accumulate_sum(&sum, src[RIDX(i, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i, j + 1, dim)]);
    accumulate_sum(&sum, src[RIDX(i - 1, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i - 1, j + 1, dim)]);
    assign_sum_to_pixel(&current_pixel, sum);
    dst[RIDX(i, j, dim)] = current_pixel;

    // bottom-right
    i = dim - 1;
    j = dim - 1;
    initialize_pixel_sum(&sum);
    accumulate_sum(&sum, src[RIDX(i, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i, j - 1, dim)]);
    accumulate_sum(&sum, src[RIDX(i - 1, j, dim)]);
    accumulate_sum(&sum, src[RIDX(i - 1, j - 1, dim)]);
    assign_sum_to_pixel(&current_pixel, sum);
    dst[RIDX(i, j, dim)] = current_pixel;

    // top
    i = 0;
    for (j = 1; j <= dim - 2; j++)
    {
        initialize_pixel_sum(&sum);
        accumulate_sum(&sum, src[RIDX(i, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j + 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j + 1, dim)]);
        assign_sum_to_pixel(&current_pixel, sum);
        dst[RIDX(i, j, dim)] = current_pixel;
    }

    // bottom
    i = dim - 1;
    for (j = 1; j <= dim - 2; j++)
    {
        initialize_pixel_sum(&sum);
        accumulate_sum(&sum, src[RIDX(i, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j + 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j + 1, dim)]);
        assign_sum_to_pixel(&current_pixel, sum);
        dst[RIDX(i, j, dim)] = current_pixel;
    }

    // left
    j = 0;
    for (i = 1; i <= dim - 2; i++)
    {
        initialize_pixel_sum(&sum);
        accumulate_sum(&sum, src[RIDX(i, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j + 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j + 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j + 1, dim)]);
        assign_sum_to_pixel(&current_pixel, sum);
        dst[RIDX(i, j, dim)] = current_pixel;
    }

    // right
    j = dim - 1;
    for (i = 1; i <= dim - 2; i++)
    {
        initialize_pixel_sum(&sum);
        accumulate_sum(&sum, src[RIDX(i, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j, dim)]);
        accumulate_sum(&sum, src[RIDX(i, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i - 1, j - 1, dim)]);
        accumulate_sum(&sum, src[RIDX(i + 1, j - 1, dim)]);
        assign_sum_to_pixel(&current_pixel, sum);
        dst[RIDX(i, j, dim)] = current_pixel;
    }

    // center
    for (i = 1; i <= dim - 2; i++)
    {
        ni = (i - 1) * dim;
        ci = i * dim;
        pi = (i + 1) * dim;
        for (j = 1; j <= dim - 2; j++)
        {
            initialize_pixel_sum(&sum);
            accumulate_sum(&sum, src[ni + j - 1]);
            accumulate_sum(&sum, src[ni + j]);
            accumulate_sum(&sum, src[ni + j + 1]);
            accumulate_sum(&sum, src[ci + j - 1]);
            accumulate_sum(&sum, src[ci + j]);
            accumulate_sum(&sum, src[ci + j + 1]);
            accumulate_sum(&sum, src[pi + j - 1]);
            accumulate_sum(&sum, src[pi + j]);
            accumulate_sum(&sum, src[pi + j + 1]);
            assign_sum_to_pixel(&current_pixel, sum);
            dst[ci + j] = current_pixel;
        }
    }
}
```

```
Smooth: Version = smooth: Current working version:
Dim		32	64	128	256	512	Mean
Your CPEs	9.8	12.4	10.9	10.5	10.6
Baseline CPEs	695.0	698.0	702.0	717.0	722.0
Speedup		71.1	56.4	64.1	68.3	68.0	65.4
```

