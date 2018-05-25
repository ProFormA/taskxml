## "Grading hints" - hinting a grader when scoring submissions

**Version 0.8**

ProFormA programming tasks can be assigned a grading scheme with the so-called *grading-hints*. This chapter describes simple and complex grading scheme examples and the XML-schema for grading-hints.

We start with a section of examples. After that we introduce the grading-hints XML schema.

### Examples

#### Introducing a simple example

A typical simple example is as follows:

```xml
<tns:grading-hints>
    <tns:root function="sum">
        <tns:test-ref ref="test1"/>
        <tns:test-ref ref="test2"/>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:root>
</tns:grading-hints>
```

A grading-hints section obtains scores from tests. Every test in a ProFormA task is expected to generate a score. If a test doesn't generate a score, it will at least generate some output like *passed* or *failed*. Such a binary output can be mapped automatically to values 1 or 0 respectively by graders that support the grading-hints schema.

The above simple example calculates the sum of all test-generated scores. From this a grader, a middle ware, or a user-connected front end like a learning management system (LMS) could produce a scoring result for students like the following:

> Your grade result is composed of several test results:
>
>   * Compilation (test1). Score achieved: 1.0 (passed)
>   
>     \<textual feedback\>
>         
>   * Unit test (test2). Score achieved: 0.45
>   
>     \<textual feedback\>
>         
>   * PMD (test3). Score achieved: 0.4 
>   
>     \<textual feedback\>
>         
>   * Checkstyle (test4). Score achieved: 0.9
>   
>     \<textual feedback\>
>         
> Total score achieved: **2.75**

The titles _Compilation_, _Unit test_, etc. are specified in the \<tests\> section of a ProFormA task. The test id printed in parentheses usually would not be included. It is only included to make our explanations comprehensible.

A tabular representation might even be clearer but lacks the possibility of including detailed feedback directly below the quantitative result. Instead a front end could provide links to detailed textual feedback:

> |         | <- Aspect    |         Score -> |               | 
> |---------|:-------------|-----------------:|--------------:|
> | _Total_ |              |                  | _Total Score_ |
> | Sum of  | Compilation  | 1.00 __details__ |               | 
> |         | Unit test    | 0.45 __details__ |               | 
> |         | PMD          | 0.40 __details__ |               | 
> |         | Checkstyle   | 0.90 __details__ |      **2.75** | 

The **details** could be clickable links that shows a popup or that routes the user to an anchor in a large HTML document.

Last but not least, a front end could combine both approaches: a tabular representation for an overview and the bullet point list for the details. The __details__ links in the table could lead the user directly to the respective section in the bullet point list.

#### Weighted sub results

Let's make the example a bit more complex by weighting test results individually:

```xml
<tns:grading-hints>
    <tns:root function="sum">
        <tns:test-ref weight="0.33" ref="test1"/>
        <tns:test-ref weight="0.20" ref="test2"/>
        <tns:test-ref weight="0.17" ref="test3"/>
        <tns:test-ref weight="0.30" ref="test4"/>
    </tns:root>
</tns:grading-hints>
```

This specifies a weighted sum out of all test-generated scores. The weights have been chosen to add up to 1.0 in order to get a weighted average. The grading-hints XML-schema does recommend but not require weights adding up to 1. 

A front end might present a result like this:

> |                 |        | <- Aspect    |         Score -> |               | 
> |-----------------|-------:|:-------------|-----------------:|--------------:|
> | _Total_         |        |              |                  | _Total Score_ |
> | Weighted sum of | x 0.33 | Compilation  | 1.00 __details__ |               | 
> |                 | x 0.20 | Unit test    | 0.45 __details__ |               | 
> |                 | x 0.17 | PMD          | 0.40 __details__ |               | 
> |                 | x 0.30 | Checkstyle   | 0.90 __details__ |      **0.76** | 

Note the additional column with factors (marked with the multiplication sign "x") and the term _Weighted Sum_ in the left column.

#### A hierarchy of sub results

Most tasks, but probably not all tasks, would consider the above configuration options to be adequate. But sometimes a task author needs more options. Consider a slightly more complicated setup that identifies different learning goals covered by different tests. E. g. test1 and test2 are considered tests that check the basic functionality of the submission, test3 and test4 are about advanced style and maintainability aspects of the solution. A task author would like to provide a global weighting scheme balancing basic and advanced aspects. Also the author would penalize errors in advanced aspects harder by taking the minimum score of test3 and test4. This would look like:

```xml
<tns:grading-hints xmlns:tns="urn:proforma:grades:v0.8">
    <tns:root function="sum">
        <tns:displaytitle>Total</tns:displaytitle>
        <tns:combine-ref weight="0.75" ref="basic"/>
        <tns:combine-ref weight="0.25" ref="advanced"/>
    </tns:root>
    <tns:combine id="basic" function="sum">
        <tns:displaytitle>Basic aspects</tns:displaytitle>
        <tns:test-ref weight="0.3" ref="test1"/>
        <tns:test-ref weight="0.7" ref="test2"/>
    </tns:combine>
    <tns:combine id="advanced" function="min">
        <tns:displaytitle>Advanced aspects</tns:displaytitle>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:combine>
</tns:grading-hints>
```

The above grading prescription defines a tree consisting of three inner nodes and four leaves.

                        +--------+
                 +------+  root  +--------+
                 |      +--------+        |
                 |                        |
         +-------+--+               +-----+----+
         |  basic   |               | advanced |
         ++-------+-+               +-+-------++
          |       |                   |       |
    +-----+-+   +-+-----+       +-----+-+   +-+-----+
    | test1 |   | test2 |       | test3 |   | test4 |
    +-------+   +-------+       +-------+   +-------+
    
The inner nodes at the medium level are ``combine`` nodes that define an accumulator function like sum, min, max or avg. Every node except the root can define a weight that will be multiplied when accumulating the result of that node to its parent node. Leave nodes are referring tests. In order to allow labeling of intermediate results at inner nodes, every inner node may get a ``displaytitle`` element. A front-end could present a result for this grading scheme in tabular form like this:

> |                 |        |                    |        | <- Aspect   |         Score -> |                           |                |
> |-----------------|-------:|--------------------|-------:|-------------|-----------------:|--------------------------:|---------------:|
> | _Total_         |        |                    |        |             |                  |                           |  _Total Score_ |
> | Weighted sum of | x 0.75 | _Basic aspects_    |        |             |                  |     _Basic aspects Score_ |                |
> |                 |        | Weighted sum of    | x 0.30 | Compilation | 1.00 __details__ |                           |                |
> |                 |        |                    | x 0.70 | Unit test   | 0.45 __details__ |                      0.62 |                |
> |                 | x 0.25 | _Advanced aspects_ |        |             |                  |  _Advanced aspects Score_ |                |
> |                 |        | Minimum of         |        | PMD         | 0.40 __details__ |                           |                |
> |                 |        |                    |        | Checkstyle  | 0.90 __details__ |                      0.40 |       **0.56** |

In a rich formatting language like HTML there would be more formatting options than in markdown. E. g. a ``colspan`` around the four cells heading east starting from _Basic aspects_ or from _Advanced aspects_ would make relationships clear. Also a ``rowspan`` around the six cells heading south starting from the upper left _Weighted sum of_ cell would work out clearly the extent of the weighted sum. An example look is given in the attached file ``gradingresult-tabular-example.png``.

#### Conditionally nullify scores

Let's build upon the previous example. A teacher or a task author wants to nullify scores for advanced aspects if the basic aspects do not exceed a certain threshold. This seems reasonable when we take a closer look at static code analysis tools that often count rule violations. A student can achieve high scores in test3 and test4 when submitting a minimal program that has near to zero functionality. For this, the task author includes a _nullify condition_ at the child reference to the ``advanced`` child:

```xml
<tns:grading-hints xmlns:tns="urn:proforma:grades:v0.8">
    <tns:root function="sum">
        <tns:displaytitle>Total</tns:displaytitle>
        <tns:combine-ref weight="0.75" ref="basic"/>
        <tns:combine-ref weight="0.25" ref="advanced">
            <tns:nullify-condition compare-op="le">
                <tns:nullify-combine-ref ref="basic"/>
                <tns:nullify-literal value="0.8"/>
            </tns:nullify-condition>
        </tns:combine-ref>
    </tns:root>
    <tns:combine id="basic" function="sum">
        <tns:displaytitle>Basic aspects</tns:displaytitle>
        <tns:test-ref weight="0.3" ref="test1"/>
        <tns:test-ref weight="0.7" ref="test2"/>
    </tns:combine>
    <tns:combine id="advanced" function="min">
        <tns:displaytitle>Advanced aspects</tns:displaytitle>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:combine>
</tns:grading-hints>
```

This way we attached a nullify condition to the tree edge between the ``root`` node and the ``advanced`` node. If the nullify condition evaluates to _true_, then the respective subresult for the subtree rooted at ``advanced`` is assumed 0. Looking at the specified condition in detail, the author requires an intermediate score of at most (``le`` = *l*ess than or *e*quals) 0.8 at the ``basic`` node. Our example submission reached 0.62, i. e. this submission would get nullified in advanced aspects.

The nullify condition extends the topology by an arrow from ``basic`` to the edge at the upper right as shown in the following graphic:

                             +--------+
               +-------------+  root  +------------+
               |             +--------+            |
               |                                   |
               |                          +-------->
               |                          |        |
         +-----+----+   nullify if <=0.8  |   +----+-----+
         |  basic   +---------------------+   | advanced |
         ++-------+-+                         +-+-------++
          |       |                             |       |
    +-----+-+   +-+-----+                 +-----+-+   +-+-----+
    | test1 |   | test2 |                 | test3 |   | test4 |
    +-------+   +-------+                 +-------+   +-------+

A possible presentation of a submission result is the following, which deviates from the previous example in the last row in the two right-most columns: 


> |                 |        |                    |        | <- Aspect   |         Score -> |                           |                |
> |-----------------|-------:|--------------------|-------:|-------------|-----------------:|--------------------------:|---------------:|
> | _Total_         |        |                    |        |             |                  |                           |  _Total Score_ |
> | Weighted sum of | x 0.75 | _Basic aspects_    |        |             |                  |     _Basic aspects Score_ |                |
> |                 |        | Weighted sum of    | x 0.30 | Compilation | 1.00 __details__ |                           |                |
> |                 |        |                    | x 0.70 | Unit test   | 0.45 __details__ |                      0.62 |                |
> |                 | x 0.25 | _Advanced aspects_ |        |             |                  |  _Advanced aspects Score_ |                |
> |                 |        | Minimum of         |        | PMD         | 0.40 __details__ |                           |                |
> |                 |        |                    |        | Checkstyle  | 0.90 __details__ |  0.40 -> 0.00 __details__ |       **0.46** |


In the cell with the _Advanced aspects_'s score the nullification could be represented by an arrow. The submitter could get a detailed explanation by clicking on _details_. The explanation could even be generated automatically like 

> When calculating the _Total Score_ your _Advanced aspects Score_ was nullified. Reason: _Basic aspects_ should be \> 0.8, but was 0.62.

The grading-hints schema allows nullification with a simple comparison like above using any of the common numerical comparators. Also an author can build composite conditions by combing simple conditions in an and-or-tree.

#### Referencing tests and sub tests

Sometimes test granularity is too coarse grained when coming to grades. From the test tool's point of view it makes sense to execute e.g. all test cases in a unit test suite at once in a single tool pass. But, in a grading scheme the various different test aspects of the individual test cases might deserve a differentiate interpretation. We build upon the above example without nullification but with a three-leveled tree. Let's assume, the unit test ``test2`` executes a test suite of two test cases. The first test case labeled ``tc.a`` is considered a basic aspect while the second test case ``tc.b`` is advanced. (Another example could concern two violation rules in a static code analysis tool, but let's stick to the unit test example so as not to complicate it.) 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tns:grading-hints xmlns:tns="urn:proforma:grades:v0.8">
    <tns:root function="sum">
        <tns:displaytitle>Total</tns:displaytitle>
        <tns:combine-ref weight="0.75" ref="basic"/>
        <tns:combine-ref weight="0.25" ref="advanced"/>
    </tns:root>
    <tns:combine id="basic" function="sum">
        <tns:displaytitle>Basic aspects</tns:displaytitle>
        <tns:test-ref weight="0.3" ref="test1"/>
        <tns:test-ref weight="0.7" ref="test2" sub-ref="tc.a">
            <tns:displaytitle>Unit test, aspect A</tns:displaytitle>
        </tns:test-ref>
    </tns:combine>
    <tns:combine id="advanced" function="min">
        <tns:displaytitle>Advanced aspects</tns:displaytitle>
        <tns:test-ref ref="test2" sub-ref="tc.b">
            <tns:displaytitle>Unit test, aspect B</tns:displaytitle>
        </tns:test-ref>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:combine>
</tns:grading-hints>
```

In the above example the task author placed ``tc.b`` below ``advanced`` while placing ``tc.a`` below ``basic``. Since the title of the \<test\> element in the ProFormA task can not be used, the author added a specific title for each referenced sub test.

The following illustrates the topology of this grading scheme:

                        +--------+
                 +------+  root  +--------+
                 |      +--------+        |
                 |                        |
         +-------+--+               +-----+-------------------+
         |  basic   |               | advanced                |
         ++-------+-+               +-+------------+--------+-+
          |       |                   |            |        |
    +-----+-+   +-+----------+   +----+-------+ +--+----+ +-+-----+
    | test1 |   | test2/tc.a |   | test2/tc.b | | test3 | | test4 |
    +-------+   +------------+   +------------+ +-------+ +-------+

The front end representation of a submission result depicts the specific sub test titles:

> |                 |        |                    |        | <- Aspect           |         Score -> |                           |                |
> |-----------------|-------:|--------------------|-------:|---------------------|-----------------:|--------------------------:|---------------:|
> | _Total_         |        |                    |        |                     |                  |                           |  _Total Score_ |
> | Weighted sum of | x 0.75 | _Basic aspects_    |        |                     |                  |     _Basic aspects Score_ |                |
> |                 |        | Weighted sum of    | x 0.30 | Compilation         | 1.00 __details__ |                           |                |
> |                 |        |                    | x 0.70 | Unit test, aspect A | 0.15 __details__ |                      0.41 |                |
> |                 | x 0.25 | _Advanced aspects_ |        |                     |                  |  _Advanced aspects Score_ |                |
> |                 |        | Minimum of         |        | Unit test, aspect B | 0.75 __details__ |                           |                |
> |                 |        |                    |        | PMD                 | 0.40 __details__ |                           |                |
> |                 |        |                    |        | Checkstyle          | 0.90 __details__ |                      0.40 |       **0.40** |
    
    
    
A caveat: The labels ``tc.a`` and ``tc.b`` are specific to the test tool. The proposed grading scheme assumes, that the test tool provides an indexed result such that when calculating grades we can randomly access each sub result. Since the grading scheme hard-codes test-tool-specific sub-result-labels, such a grading scheme has a lower chance to be exchanged between graders, especially between different test tool backends.

#### Combining sub tests and nullify conditions

An author can combine sub test references and test references. As an example we extend the previous example. The author wants to nullify the score from the compilation test when all unit test cases miss the bar "0.5". For this in the following example the author adds one additional ``combine`` node labeld ``test2.max`` expressing the maximum of all unit test cases. With the new ``test2.max`` node it is easy to nullify the ``test1`` result, if the ``test2.max`` value is less than 0.5:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tns:grading-hints xmlns:tns="urn:proforma:grades:v0.8">
    <tns:root function="sum">
        <tns:displaytitle>Total</tns:displaytitle>
        <tns:combine-ref weight="0.75" ref="basic"/>
        <tns:combine-ref weight="0.25" ref="advanced"/>
    </tns:root>
    <tns:combine id="basic" function="sum">
        <tns:displaytitle>Basic aspects</tns:displaytitle>
        <tns:test-ref weight="0.3" ref="test1">
            <tns:nullify-condition compare-op="lt">
                <tns:nullify-combine-ref ref="test2.max"/>
                <tns:nullify-literal value="0.5"/>
            </tns:nullify-condition>
        </tns:test-ref>
        <tns:test-ref weight="0.7" ref="test2" sub-ref="tc.a">
            <tns:displaytitle>Unit test, aspect A</tns:displaytitle>
        </tns:test-ref>
    </tns:combine>
    <tns:combine id="advanced" function="min">
        <tns:displaytitle>Advanced aspects</tns:displaytitle>
        <tns:test-ref ref="test2" sub-ref="tc.b">
            <tns:displaytitle>Unit test, aspect B</tns:displaytitle>
        </tns:test-ref>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:combine>
    <tns:combine id="test2.max" function="max">
        <tns:displaytitle>Best result of all unit test aspects</tns:displaytitle>
        <tns:test-ref ref="test2" sub-ref="tc.a"/>
        <tns:test-ref ref="test2" sub-ref="tc.b"/>
    </tns:combine>
</tns:grading-hints>
```

The topology shows the following graphic. There is a node ``test.max`` not connected to the root. The only purpose of the tree rooted at ``test2.max`` is a nullification condition of the edge between ``basic`` and ``test1``.

                                                             +--------+
                                                      +------+  root  +--------+
                                                      |      +--------+        |
                                                      |                        |
          +-----------+   nullify if <0.5     +-------+--+               +-----+-------------------+
          | test2.max +----------+            |  basic   |               | advanced                |
          ++-------+--+          |            ++-------+-+               +-+------------+--------+-+
           |       |             |             |       |                   |            |        |
    +------+---+  ++---------+   +------------->       |                   |            |        |
    |test2/tc.a|  |test2/tc.b|                 |       |                   |            |        |
    +----------+  +----------+           +-----+-+   +-+----------+   +----+-------+ +--+----+ +-+-----+
                                         | test1 |   | test2/tc.a |   | test2/tc.b | | test3 | | test4 |
                                         +-------+   +------------+   +------------+ +-------+ +-------+



A possible representation in a front end is as follows. It does deviate from the previous one in a single cell at row 4, column 6:

> |                 |        |                    |        | <- Aspect           |                 Score -> |                           |                |
> |-----------------|-------:|--------------------|-------:|---------------------|-------------------------:|--------------------------:|---------------:|
> | _Total_         |        |                    |        |                     |                          |                           |  _Total Score_ |
> | Weighted sum of | x 0.75 | _Basic aspects_    |        |                     |                          |     _Basic aspects Score_ |                |
> |                 |        | Weighted sum of    | x 0.30 | Compilation         | 1.00 -> 1.00 __details__ |                           |                |
> |                 |        |                    | x 0.70 | Unit test, aspect A |         0.15 __details__ |                      0.41 |                |
> |                 | x 0.25 | _Advanced aspects_ |        |                     |                          |  _Advanced aspects Score_ |                |
> |                 |        | Minimum of         |        | Unit test, aspect B |         0.75 __details__ |                           |                |
> |                 |        |                    |        | PMD                 |         0.40 __details__ |                           |                |
> |                 |        |                    |        | Checkstyle          |         0.90 __details__ |                      0.40 |       **0.40** |

The Compilation Score does not get nullified, because the maximum of tc.a and tc.b scores is \>= 0.5. The user could get the following,automatically generated explanation when clicking on the __details__ link besides the "1.00 -\> 1.00" mark:

> When calculating the _Basic aspects Score_ your _Compilation Score_ was not nullified. Reason: _Best result of all unit test aspects_ should be \>= 0.5 and was 0.75.

If deemed necessary, the explanation could be extended by an extra table that explains the calculation of ``test2.max``'s score, but usually a carefully chosen display title for the ``test.max`` node might be less confusing.

By the way: the same effect of nullifying the compilation score when the best unit test result is less than 0.5 could have been reached by a composite nullify condition.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<tns:grading-hints xmlns:tns="urn:proforma:grades:v0.8">
    <tns:root function="sum">
        <tns:displaytitle>Total</tns:displaytitle>
        <tns:combine-ref weight="0.75" ref="basic"/>
        <tns:combine-ref weight="0.25" ref="advanced"/>
    </tns:root>
    <tns:combine id="basic" function="sum">
        <tns:displaytitle>Basic aspects</tns:displaytitle>
        <tns:test-ref weight="0.3" ref="test1">
            <tns:nullify-conditions compose-op="and">
                <tns:displaytitle>Compilation score gets nullified when all unit tests miss 0.5</tns:displaytitle>
                <tns:nullify-condition compare-op="lt">
                    <tns:nullify-test-ref ref="test2" sub-ref="tc.a"/>
                    <tns:nullify-literal value="0.5"/>
                </tns:nullify-condition>
                <tns:nullify-condition compare-op="lt">
                    <tns:nullify-test-ref ref="test2" sub-ref="tc.b"/>
                    <tns:nullify-literal value="0.5"/>
                </tns:nullify-condition>
            </tns:nullify-conditions>
        </tns:test-ref>
        <tns:test-ref weight="0.7" ref="test2" sub-ref="tc.a">
            <tns:displaytitle>Unit test, aspect A</tns:displaytitle>
        </tns:test-ref>
    </tns:combine>
    <tns:combine id="advanced" function="min">
        <tns:displaytitle>Advanced aspects</tns:displaytitle>
        <tns:test-ref ref="test2" sub-ref="tc.b">
            <tns:displaytitle>Unit test, aspect B</tns:displaytitle>
        </tns:test-ref>
        <tns:test-ref ref="test3"/>
        <tns:test-ref ref="test4"/>
    </tns:combine>
</tns:grading-hints>
```

This time the user could get the following detail information, thtat can be generated automatically from the grading scheme:

> **Compilation score gets nullified when all unit tests miss 0.5**
>
> When calculating the _Basic aspects Score_ your _Compilation Score_ was __not__ nullified. 
> 
> Reason: At least one of the following conditions was True:
>
>   - _Unit test, aspect A_ should be \>= 0.5 and was 0.15.
>   - _Unit test, aspect B_ should be \>= 0.5 and was 0.75.

The advantage of this approach is that of saving on an additional tree node. It depends on the task and the condition, whether an additional (reusable) tree node is better suited or the (nestable) composite nullify conditions. 

#### Last: a very simple example

Having seen so manycomplicated examples, we should state, that most tasks do not need to specify anything complex. The following is the simplest grading-hints element possible:

```xml
<tns:grading-hints>
    <tns:root/>
</tns:grading-hints>
```

When calculating grades based in these simple grading-hints \<test\> element's scores are obtained and the minimum score will be the result.

### XML schema

to be continued ...



