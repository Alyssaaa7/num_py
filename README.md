# numc

Here's what we did in project 4:
- In Project 4, we spent a few days reading over the spec and trying to understand how the different parts of the project fit together. Our numc module consists of two main C files, matrix.c and numc.c, which are setup into a Python module using the setup.py file. The file matrix.c contains several helper functions, including add, subtract, negate, absolute value, multiply, and power, along with the functions for allocating a new matrix and allocating a matrix that uses a slice of a previously formed matrix. We first completed the add, subtract, absolute value, and negation functions, which had relatively simple logic, but we spent some time on the allocate, deallocate, multiply, and power functions, trying to, first of all, understand C pointers better, and then how to implement the different matrices within the framework provided. Our allocate function allocates space first for the new matrix structure, then makes space for the data. Initially, our data was in 2D, where we first memory allocated an array of pointers, and then went into each pointer and calloced space for a row of the required number of columns for each pointer in that array. However, we realized later on that this was inefficient in terms of always requiring two for loops to access each double value in the data and in terms of our rows not being continuous in memory, so we switched the data to be a 1D array of the size the matrix, and then assigned pointers to each row to make it seem like a pseudo-2D data array for our matrix. However, we still have a double pointer because other matrices' data might use the same data, and thus that would make it easier to access. Thus, we went back and changed all of our simple functions to run in one loop across all of the elements of this rows * cols size data, which simplified our code a little. Allocating matrices using references was similar, except instead of allocating space for a new data array, we allocated just an array of pointers for each row, and then set those pointers to the parent's data with offsets added to the pointer. Thus, both matrices pointed to the same data overall, but a child would have its rows and columns starting at different offsets. Even if the sizes weren't the same, each matrix knows how large it should be, so that would always be a stopping point. In this way, editing any of the matrices that are children of a matrix causes the original matrix to change as well, as specified. We spent the most time on our power function in the first task, because we could not use a matrix as both something to be multiplied and somewhere where the solution should go based on the nature of C pointers and how they point to the same data. This would have caused incorrect values to be used in matrix multiplication, whose naive formula we used an implemention of from our class lectures, which was coded primarily by Xinyu. However, we had issues when we had not set the result matrix to be filled with zeros everytime, which took some extra computation, but allowed us to meet the required specifications. Thus, after many efforts, Xinyu finally came up with a stratgey where we allocate a temporary matrix to be used in the duration of the power function, which stores intermediate values in different matrices depending on whether the current power is odd or even, and then the final result is swapped to be the temporary matrix. We were manually changing the values of the result matrix for a long time, but then we realized we could just swap their data pointers, which is what we tried doing. Overall, Task 1 took longer than we had expected, but taught us a lot about C pointers and making the most of memory space. Both Xinyu and Anvitha worked on functions together, discussing implementions and then reading and correcting each others' code so that we both understood what our code was doing.
- After implementing the functions in matrix.c with the correct functionality, Anvitha worked on Task 2, which essentially involved writing two functions, Extension and setup, using the Python-C documentation, which just included matching the correct parameters within the function call and then calling the function. After this, we were able to import our numc module into a Python environment (which had to be a virtual environment because Python is read-only locally) and create matrices. 
- For Task 3, it took us a long time to understand the nuances of the Python-C interface, since the documentation was not always clear on what was allowed and what was not allowed, as well as the differences between certain types and how they could be used, but after some attempts and fixing syntax issues in Matrix61c_add, implementing the rest of the numerical functions was mostly similar with slightly different error checking and function calls to the helper functions we had written in matrix.c. Anvitha coded all of the number methods in numc.c, as well as implemented Matrix61c_as_number, but Xinyu suggested simplifications to error checking that would reduce the amount of computation, even though they would provide less specific error messages to a user. In a similar fashion, Anvitha coded the get and set methods in numc.c, which were reviewed by Xinyu afterwards. While set and get were similar in implementation to the number methods, with several error checks and then a call to the complementary method in matrix.c, understanding how to parse and unpack a tuple took the most time for these methods, since error checks and exact functionality did not always seem very clear. The longest part of Task 3 was implementing subscript and set_subscript, which allow for slicing functionality in the numc module, as well as for changing only the values contained in slices. After much discussion and drawing out cases, Xinyu coded the functions for subscript and set_subscript, which thoroughly and carefully checked each and every possible case and were modularized using if statements and which took several hours, and then Anvitha spent a few hours the next day reviewing the code and checking and simplifying all of the logic, bringing the functions down to around 200 lines of code each. After implementing these functions, we imported our numc module and ran different pieces of code to ensure that our framework covered all of the specifications. These tests were mainly for correctness, and we were able to find one edge case which wasn't caught and resulted in a segmentation fault, which we fixed. 
- Before utilizing more sophisticated methods for optimizing our code, Anvitha and Xinyu first went through all of our functions and simplified logic, reduced mmemory accesses by making temporary variables, and condensed as many for loops as possible without compromising functionality. With these naive methods, our speedup for functions was around 1-3x. Then, Anvitha worked on using OpenMP to speed up the simple functions within matrix.c, which included adding a #pragma omp parallel for statement before the for loops in add, subtract, negate, and absolute value. However, this seemed to slow down our implementation, so we added an if condition to only use OpenMP on computations with matrices larger than 1000x1000, which sped it up. We tried to speedup our framework in as simple ways as possible, so the next action we took was to unroll loops for those same simple functions, and included a case for tail cases at the end. This allowed us to do four computations per loop in the for loop, and decreased the number of times we were looping. With these two measures, along with reducing memory accesses by creating temporary pointers, we were able to reach the benchmark of 4.9x speedup for the simple functions. Xinyu also tried implement simd for multiply, but it seems not work well for speed up the matrix multiplication. First, Xinyu tried to use na•ve approach to use simd and unrolling for matrix2Õs columns, and the speedup as 30-40 at that time. Then, Xinyu also tried to add a transpose function, which transpose matrix 2. We thought this wll increase our performance, but it looks like it doesn't help. We may just want to use naive approach with loop unrolling and pragma.

Functions in Numc:
Once you import your numc module, you can use it to create matrices of different sizes in different ways, such as:
- numc.Matrix(3, 3) # This creates a 3 * 3 matrix with entries all zeros
- numc.Matrix(3, 3, 1) # This creates a 3 * 3 matrix with entries all ones
- numc.Matrix([1, 2, 3], [4, 5, 6]) # This creates a 2 * 3 matrix with first row 1, 2, 3, second row 4, 5, 6
- numc.Matrix(1, 2, [4, 5]) # This creates a 1 * 2 matrix with entries 4, 5
(The above code is straight from the project specifications.)

If matrixes are declared, and assigned to variables, such as:
A = numc.Matrix(3, 3)
B = numc.Matrix(3, 3)

Element-Wise Addition: 
A + B
This returns a new matrix with its elements being the sum of the elements of the two matrices.

Element-Wise Subtraction:
A - B
This returns a new matrix with its elements being the difference of the elements of the two matrices.

Matrix Multiplication:
A * B
This returns a new matrix holding the matrix multiplication of A and B. All conditions for matrix multiplication must be met.

Element-Wise Negation:
-A 
This returns a new matrix with all the elements being the negated values of the elements of A.

Absolute Value:
abs(A)
This returns a new matrix with all the elements being the absolute value of the elements of A.

Power:
A ** pow
This multiplies matrix A by itself, pow number of times. Pow must be an integer greater than 0.

Our numc implementation also allows for slicing. Slicing can be done using A[start_row:end_row, start_column:end_column]. Step size can also be changed in this implementation, but our numc allows only for step sizes of one. Slices of a matrix can also be set to new values, but an appropriate value must be replacing it, whether that is an integer, a 1D array, or a 2D array.

A.set(row, column, value) and A.get(row, column) can also be used to set specific values of a matrix or get the value at a certain row and column.
