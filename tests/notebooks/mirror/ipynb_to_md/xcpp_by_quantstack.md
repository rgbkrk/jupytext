---
jupyter:
  kernelspec:
    display_name: C++14
    language: C++14
    name: xeus-cling-cpp14
---

[![xeus-cling](images/xeus-cling.png)](https://github.com/QuantStack/xeus-cling/)

A Jupyter kernel for C++ based on the `cling` C++ interpreter and the `xeus` native implementation of the Jupyter protocol, xeus.

- GitHub repository: https://github.com/QuantStack/xeus-cling/
- Online documentation: https://xeus-cling.readthedocs.io/


## Usage

<div style="background: #efffed;
            border: 1px solid grey;
            margin: 8px 0 8px 0;
            text-align: center;
            padding: 8px; ">
    <i class="fa-play fa" 
       style="font-size: 40px;
              line-height: 40px;
              margin: 8px;
              color: #444;">
    </i>
    <div>
    To run the selected code cell, hit <pre style="background: #efffed">Shift + Enter</pre>
    </div>
</div>


## Output and error streams

`std::cout` and `std::cerr` are redirected to the notebook frontend.

```c++
#include <iostream>

std::cout << "some output" << std::endl;
```

```c++
std::cerr << "some error" << std::endl;
```

```c++
#include <stdexcept>
```

```c++
throw std::runtime_error("Unknown exception");
```

Omitting the `;` in the last statement of a cell results in an output being printed

```c++
int j = 5;
```

```c++
j
```

# Interpreting the C++ programming language

`cling` has a broad support of the features of C++. You can define functions, classes, templates, etc ...


## Functions

```c++
double sqr(double a)
{
    return a * a;
}
```

```c++
double a = 2.5;
double asqr = sqr(a);
asqr
```

## Classes

```c++
class Foo
{
public:

    virtual ~Foo() {}
    
    virtual void print(double value) const
    {
        std::cout << "Foo value = " << value << std::endl;
    }
};
```

```c++
Foo bar;
bar.print(1.2);
```

## Polymorphism

```c++
class Bar : public Foo
{
public:

    virtual ~Bar() {}
    
    virtual void print(double value) const
    {
        std::cout << "Bar value = " << 2 * value << std::endl;
    }
};
```

```c++
Foo* bar2 = new Bar;
bar2->print(1.2);
delete bar2;
```

## Templates

```c++
#include <typeinfo>

template <class T>
class FooT
{
public:
    
    explicit FooT(const T& t) : m_t(t) {}
    
    void print() const
    {
        std::cout << typeid(T).name() << " m_t = " << m_t << std::endl;
    }
    
private:
    
    T m_t;
};

template <>
class FooT<int>
{
public:
    
    explicit FooT(const int& t) : m_t(t) {}
    
    void print() const
    {
        std::cout << "m_t = " << m_t << std::endl;
    }
    
private:
    
    int m_t;
};
```

```c++
FooT<double> foot1(1.2);
foot1.print();
```

```c++
FooT<int> foot2(4);
foot2.print();
```

## C++11 / C++14 support

```c++
class Foo11
{
public:
    
    Foo11() { std::cout << "Foo11 default constructor" << std::endl; }
    Foo11(const Foo11&) { std::cout << "Foo11 copy constructor" << std::endl; }
    Foo11(Foo11&&) { std::cout << "Foo11 move constructor" << std::endl; }
};
```

```c++
Foo11 f1;
Foo11 f2(f1);
Foo11 f3(std::move(f1));
```

```c++
#include <vector>

std::vector<int> v = { 1, 2, 3};
auto iter = ++v.begin();
v
```

```c++
*iter
```

... and also lambda, universal references, `decltype`, etc ...


## Documentation and completion

 - Documentation for types of the standard library is retrieved on cppreference.com.
 - The quick-help feature can also be enabled for user-defined types and third-party libraries. More documentation on this feature is available at https://xeus-cling.readthedocs.io/en/latest/inline_help.html.


```c++
?std::vector
```

## Using the `display_data` mechanism


For a user-defined type `T`, the rich rendering in the notebook and JupyterLab can be enabled by by implementing the function `xeus::xjson mime_bundle_repr(const T& im)`, which returns the JSON mime bundle for that type.

More documentation on the rich display system of Jupyter and Xeus-cling is available at https://xeus-cling.readthedocs.io/en/latest/rich_display.html


### Image example

```c++
#include <string>
#include <fstream>

#include "xtl/xbase64.hpp"
#include "xeus/xjson.hpp"

namespace im
{
    struct image
    {   
        inline image(const std::string& filename)
        {
            std::ifstream fin(filename, std::ios::binary);   
            m_buffer << fin.rdbuf();
        }
        
        std::stringstream m_buffer;
    };
    
    xeus::xjson mime_bundle_repr(const image& i)
    {
        auto bundle = xeus::xjson::object();
        bundle["image/png"] = xtl::base64encode(i.m_buffer.str());
        return bundle;
    }
}
```

```c++
im::image marie("images/marie.png");
marie
```

### Audio example

```c++
#include <string>
#include <fstream>

#include "xtl/xbase64.hpp"
#include "xeus/xjson.hpp"

namespace au
{
    struct audio
    {   
        inline audio(const std::string& filename)
        {
            std::ifstream fin(filename, std::ios::binary);   
            m_buffer << fin.rdbuf();
        }
        
        std::stringstream m_buffer;
    };
    
    xeus::xjson mime_bundle_repr(const audio& a)
    {
        auto bundle = xeus::xjson::object();
        bundle["text/html"] =
           std::string("<audio controls=\"controls\"><source src=\"data:audio/wav;base64,")
           + xtl::base64encode(a.m_buffer.str()) +
            "\" type=\"audio/wav\" /></audio>";
        return bundle;
    }
}
```

```c++
au::audio drums("audio/audio.wav");
drums
```

### Display

```c++
#include "xcpp/xdisplay.hpp"
```

```c++
xcpp::display(drums);
```

### Update-display

```c++
#include <string>
#include "xcpp/xdisplay.hpp"

namespace ht
{
    struct html
    {   
        inline html(const std::string& content)
        {
            m_content = content;
        }
        std::string m_content;
    };

    xeus::xjson mime_bundle_repr(const html& a)
    {
        auto bundle = xeus::xjson::object();
        bundle["text/html"] = a.m_content;
        return bundle;
    }
}

// A red rectangle
ht::html rect(R"(
<div style='
    width: 90px;
    height: 50px;
    line-height: 50px;
    background-color: blue;
    color: white;
    text-align: center;'>
Original
</div>)");
```

```c++
xcpp::display(rect, "some_display_id");
```

```c++
// Update the rectangle to be blue
rect.m_content = R"(
<div style='
    width: 90px;
    height: 50px;
    line-height: 50px;
    background-color: red;
    color: white;
    text-align: center;'>
Updated
</div>)";

xcpp::display(rect, "some_display_id", true);
```

## Magics

Magics are special commands for the kernel that are not part of the C++ language.

They are defined with the symbol `%` for a line magic and `%%` for a cell magic.

More documentation for magics is available at https://xeus-cling.readthedocs.io/en/latest/magics.html.

```c++
#include <algorithm>
#include <vector>
```

```c++
std::vector<double> to_shuffle = {1, 2, 3, 4};
```

```c++
%timeit std::random_shuffle(to_shuffle.begin(), to_shuffle.end());
```

[![xtensor](images/xtensor.png)](https://github.com/QuantStack/xtensor/)

- GitHub repository: https://github.com/QuantStack/xtensor/
- Online documentation: https://xtensor.readthedocs.io/
- NumPy to xtensor cheat sheet: http://xtensor.readthedocs.io/en/latest/numpy.html

`xtensor` is a C++ library for manipulating N-D arrays with an API very similar to that of numpy.

```c++
#include <iostream>

#include "xtensor/xarray.hpp"
#include "xtensor/xio.hpp"
#include "xtensor/xview.hpp"

xt::xarray<double> arr1
  {{1.0, 2.0, 3.0},
   {2.0, 5.0, 7.0},
   {2.0, 5.0, 7.0}};

xt::xarray<double> arr2
  {5.0, 6.0, 7.0};

xt::view(arr1, 1) + arr2
```

Together with the C++ Jupyter kernel, `xtensor` offers a similar experience as `NumPy` in the Python Jupyter kernel, including broadcasting and universal functions.

```c++
#include <iostream>
#include "xtensor/xarray.hpp"
#include "xtensor/xio.hpp"
```

```c++
xt::xarray<int> arr
  {1, 2, 3, 4, 5, 6, 7, 8, 9};

arr.reshape({3, 3});

std::cout << arr;
```

```c++
#include "xtensor-blas/xlinalg.hpp"
```

```c++
xt::xtensor<double, 2> m = {{1.5, 0.5}, {0.7, 1.0}};
std::cout << "Matrix rank: " << std::endl << xt::linalg::matrix_rank(m) << std::endl;
std::cout << "Matrix inverse: " << std::endl << xt::linalg::inv(m) << std::endl;
std::cout << "Eigen values: " << std::endl << xt::linalg::eigvals(m) << std::endl;
```

```c++
xt::xarray<double> arg1 = xt::arange<double>(9);
xt::xarray<double> arg2 = xt::arange<double>(18);

arg1.reshape({3, 3});
arg2.reshape({2, 3, 3});

std::cout << xt::linalg::dot(arg1, arg2) << std::endl;
```

```c++

```
