---
layout: post
title: MSSQL support for PHTF
---

I'd been working on a project that required a MS SQL database, so I developed a small class to connect to this database engine.
There are a few DB drivers for MSSQL, but you should use a windows package named SQL native driver (sqldrv) whit PDO support. It's way better than old (and deprecated) mssql_* Php functions. 
You can check this improvements in [Php Transparent Framework](https://github.com/comtom/phtf).

Cheers!

