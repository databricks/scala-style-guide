# Databricks Scala Guide 

With over 1000 contributors, Apache Spark is to the best of our knowledge the largest open-source project in Big Data and the most active project written in Scala. This guide draws from our experience coaching and working with engineers contributing to Spark as well as our [Databricks](http://databricks.com/) engineering team.

Code is __written once__ by its author, but __read and modified multiple times__ by lots of other engineers. As most bugs actually come from future modification of the code, we need to optimize our codebase for long-term, global readability and maintainability. The best way to achieve this is to write simple code.

Scala is an incredibly powerful language that is capable of many paradigms. We have found that the following guidelines work well for us on projects with high velocity. Depending on the needs of your team, your mileage might vary.

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.


## <a name='TOC'>Table of Contents</a>

[Document History](history)
[Syntactic Style](syntactic)
[Scala Language Features](lang)
[Concurrency](concurrency)
[Java Interoperability](java)
[Testing](testing)
[Miscellaneous](misc)



## <a name='history'>Document History</a>
- 2015-03-16: Initial version.
- 2015-05-25: Added [override Modifier](#override_modifier) section.
- 2015-08-23: Downgraded the severity of some rules from "do NOT" to "avoid".
- 2015-11-17: Updated [apply Method](#apply_method) section: apply method in companion object should return the companion class.
- 2015-11-17: This guide has been [translated into Chinese](README-ZH.md). The Chinese translation is contributed by community member [Hawstein](https://github.com/Hawstein). We do not guarantee that it will always be kept up-to-date.
- 2015-12-14:  This guide has been [translated into Korean](README-KO.md). The Korean translation is contributed by [Hyukjin Kwon](https://github.com/HyukjinKwon) and reviewed by [Yun Park](https://github.com/yunpark93), [Kevin (Sangwoo) Kim](https://github.com/swkimme), [Hyunje Jo](https://github.com/RetrieverJo) and [Woochel Choi](https://github.com/socialpercon). We do not guarantee that it will always be kept up-to-date.
- 2016-06-15: Added [Anonymous Methods](#anonymous) section.
- 2016-06-21: Added [Variable Naming Convention](#variable-naming) section.
- 2016-12-24: Added [Case Classes and Immutability](#case_class_immutability) section.
- 2017-02-23: Added [Testing](#testing) section.
- 2017-04-18: Added [Prefer existing well-tested methods over reinventing the wheel](#misc_well_tested_method) section.

