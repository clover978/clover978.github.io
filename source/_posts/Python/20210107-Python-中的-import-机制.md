---
title: Python 中的 import 机制
date: 2021-01-07 10:56:12
tags:
  - Python
categories:
  - Python
---

- module import 
```Python
+ A/
    - a.py
        ```
        import m
        from m import *
        ```
    - m.py
        ```
        # empty
        ```
        
'''
最简单的 import 情况
'''
```
<!-- more -->

- package import
```Python
+ A/
    - a.py
        ```
        import m
        from m import *
        
        import C
        from C import *
        ```
    - m.py
        ```
        import C
        from C import *
        ```
    + C/
        - __init__.py
            ```
            # empty
            ```
        
'''
package 的 import

  Q - ImportError: No module named C
  A : python2 中，package 文件夹下必须有 __init__.py
'''
```

- relative import
```bash
##################################################################################
    'import 关键字的 相对导入'
+ X/
    + A/
        - __init__.py
        - a.py
            ```
            import m
            from m import *
            
            import C
            from C import *
            ```
        - m.py
            ```
            import C
            from C import *
            ```            
'''
在 A 的 父级目录执行 python -c "import A.a"

    a.py:1 --> import m
    Q - ModuleNotFoundError: No module named 'm'
    A : import <pkg/module>， pkg/module 必须在 PYTHONAPTH 路径下，所以这里有两种解决方案，
        一种是将目录 A 的路径加入 PYTHONPATH；一种是改成 import A.m。  
        很明显后一种改法固定了调用脚本与 packageA 的相对路径关系，因此只有在 A 作为 X 的一个 subpackage 才可以这么写；而 PYTHONPATH 又跟代码所在的路径相关，因此也不太实用。
        所以这种情况下就需要用 from <> import <> 语法，这种语法支持 relative import
'''


###################################################################################
    'from ... import ... 的相对导入'
+ X/
    + A/
        - __init__.py
        - a.py
            ```
            # import A.m
            from m import *
            
            # import A.C
            from C import *
            ```
        - m.py
            ```
            # import A.C
            from C import *
            ```    
        + C/
'''
修正好 import m/C 的错误之后，继续执行 python -c "import A.a"

    a.py:1 --> import A.m
    m.py:2 -->  from C import *
    Q - ModuleNotFoundError: No module named 'm'
    A : 这里可以通过改 PYPATHPATH 消除这个错误；
        如果不改环境变量的话，from ... import ... 语法支持相对导入，改成 
            from .C import *    # 这里的 .C 是相对于 __package__ 变量的, 这一句等价于下面
            from __package__.C    # 由于执行的语句是 import A.m， 所以这里的 __package__='A'，所以等价于下面
            from A.C import *
'''


##################################################################################
    'from ... import ... 的导入(2)'
+ X/
    + A/
        - __init__.py
        - a.py
            ```
            # import A.m
            from .m import *
            
            # import A.C
            from .C import *
            ```
        - m.py
            ```
            # import A.C
            from .C import *
            ```   
        + C/
    + B/
        - __init__.py
        - b.py
            ```
            from ..A import *
            ```
'''
将 A 中的导入全部都变成相对导入之后，试一下在 B 里面导入 package A，
执行 python -c "import B.b"

    --- b.py:1 --> from ..A import *
    Q - ValueError: attempted relative import beyond top-level package
    A : 这是由于 b.py 中的 import 语句使用了上一级的 package，而调用语句是 import B.b，只有一层 package，
        要避免这个错误，需要将 X 作为一个 package 运行： 
            python -c "import X.B.b"
'''


###################################################################################
    'module 的执行'
+ X/
    + A/
        - __init__.py
        - a.py
            ```
            # import A.m
            from .m import *
            
            # import A.C
            from .C import *
            ```
        - m.py
            ```
            # import A.C
            from .C import *
            ```   
        + C/
    + B/
        - __init__.py
        - b.py
            ```
            from ..A import *
            ```
'''
先总结一下，当你实现一个模块 a.py 后，如果你想在其他路径下写代码调用这个模块，就会用到形如 import A.a 这样的语句，
但是这种情况下，a.py 当中的 import 语句可能会报错。
因为 import 语句只支持绝对导入，所以这种情况要用 from .. import ..，并且将其改成相对导入的形式。

但是这种情况下，你再回到文件夹 A，却无法执行 python a.py 了

    --- a.py:2 --> from .m import *
    Q - ModuleNotFoundError: No module named '__main__.m'; '__main__' is not a package
    A : 执行 import A.a 的时候，a 作为一个 module，from 语句从 __package__ 变量相对的路径下去 import
        执行 python A/a.py 的时候， a 作为一个 脚本，__package__=None，from 语句从 __name__ 的相对路径下去 import
    
这时正确的做法是将 a.py 作为一个 module 执行。
    python [-i] -m A.a    # 用了这么久才知道 python 有个 -i 选项 !!!!!!!!
'''

```
