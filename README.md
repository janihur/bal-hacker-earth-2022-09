# Ballerina Coding Challenge

[Ballerina](https://ballerina.io/) [Coding Challenge](https://www.hackerearth.com/challenges/competitive/ballerina-coding-challenge/) in [hackerearth.com](https://www.hackerearth.com/challenges/) September 2022. Uses Ballerina [Swan Lake Update 2](https://ballerina.io/downloads/swan-lake-release-notes/swan-lake-2201.2.0).

## Notes About Hacker Earth

I had serious issues with Hacker Earth website. In fact so serious that I gave up wasting my time there. They have web IDE that should be used to edit the code, run tests and submit the solution but it was slow at best and it constantly failed to load - either it just became non-responsive for tens of minutes and then I gave up or returned 504 gateway time-out. E.g. I was able to launch the web IDE once to download the problem #2 tar-file but after that I never succeeded to load the web IDE second time to upload the solution.

The web IDE has no support for Ballerina but it was just a bloated text editor running in a browser. I used the IDE only to download the problem tar-files and to upload the solution. All work was done with locally installed VS Code with Ballerina plugin. This was also the workflow recommended by WSO2 (that organizes the event).

You can guess I'm not a fan of Hacker Earth and I won't recommend it at the moment. That might change if they could improve the performance of the site. Anyway the first time charm was destroyed and I'm not sure if I'm going to give them a second change.

## Notes About The Problems

Those few problems that I was able to work with were pretty much ok. The challenge level was in [Ballerina By Example](https://ballerina.io/learn/by-example/) cathegory so if you have ever done programming you should have not that much problems. The main point here was not the competition but to learn how to use Ballerina features.

### Problem #2

In problem #2 [type inclusion](https://ballerina.io/learn/by-example/type-inclusion-for-records/) unexpectedly changes the column ordering in the output CSV file (`io:fileWriteCsv()`).

I.e.

```
type BaseFillUp record {|
    readonly int employee_id;
    int gas_fill_up_count;
    decimal total_fuel_cost_usd;
    decimal total_gallons;
|};

type TotalFillUp record {|
    *BaseFillUp;
    int total_miles_accrued;
|};
```

and

```
type TotalFillUp record {|
    readonly int employee_id;
    int gas_fill_up_count;
    decimal total_fuel_cost_usd;
    decimal total_gallons;
    int total_miles_accrued;
|};
```

that should be identical data structures create a different CSV rule when passed to `io:fileWriteCsv()`.

This was later [confirmed](https://stackoverflow.com/q/73812379/272735) by WSO2 to be a [a standard library bug](https://github.com/ballerina-platform/ballerina-standard-library/issues/3399).

### Problem #4

In problem #4 the answer is only accepted if the insignificant whitespace of the output XML file matches. I.e.

```
<foo>
  <bar>BAR</bar>
<foo>
```

doesn't match

```
<foo><bar>BAR</bar><foo>
```

even they should match if you consider them as XML and not strings. At the moment I'm not aware of simple way to remove the insignificant whitespace.

### Problem #5

Problem #5 uses [H2 database](http://www.h2database.com/). When I was able to submit the solution I got only 50% of the test cases right and my later submissions where bluntly time-outed. Anyway that made me wonder how to peek inside the database. I found [H2 Console](http://www.h2database.com/html/tutorial.html#tutorial_starting_h2_console) that can be launched by running H2 jar-file, e.g. in my case:

```
$ java -jar target/platform-libs/com/h2database/h2/2.0.206/h2-2.0.206.jar &
```

I had there a major problem as I didn't saw the pre-created tables in the database even my Ballerina code was able to access them.

I finally figured out that:

* if database file is `/tmp/problem_2_1/db/gofigure.mv.db`
* the CORRECT jdbc url is: `jdbc:h2:file:/tmp/problem_2_1/db/gofigure`
* and NOT: `jdbc:h2:file:/tmp/problem_2_1/db/gofigure.mv.db` (H2 Console kindly creates a new empty database file `gofigure.mv.db.mv.db`)

Go figure!

Later I realised I had to forgot to update `Ballerina.toml` in the web IDE too. After that my original submisson was 100%.

### Problem #7

In problem #7 I couldn't get H2 SQL join syntax right when writing the query in HS Console. E.g. query that looks a perfectly valid:

```
select e.name, e.department, p.amount, p.reason
from employee e
inner join payment p 
on e.employee_id = p.employee_id
where p.amount > 3000
order by p.amount
;
```

Gave `org/h2/table/TableFilter$MapColumnsVisitor` error with useless Java stack trace but e.g.

```
select e.name, e.department
from employee as e
;
```

works as expected. The big surprise here that despite that problem the query works from Ballerina code! Beats me - at the moment I blame H2 Console but I might be wrong. Anyway I'd like to see here the industry standard embedded SQLite database that has a working CLI instead of clunky H2.

## Summary

Hacker Earth sucks.

WSO2 members of the Ballerina community were active and responsive as always.

I also side-tracked several times based on Discord discussions but that's part of the deal here!

Personally it was very disappointing I could submit solutions only for 6/20 essentially hello-world type challenges. Problems with Hacker Earth, one Ballerina bug and issues with H2 Console killed the flow totally. Even I can't say this was a positive experience I'm waiting eagerly the next Ballerina coding challenge!

My only concern about Ballerina is that I keep running into genuine or well known bugs constanly even with a quite trivial Ballerina code. Of course one have to consider that Ballerina is not only the programming language but also the large standard library so there is lot of shadows for the bugs. I just hope we don't see regression in [nBallerina](https://github.com/ballerina-platform/nballerina#readme) (that will be the next generation implementation of Ballerina). But on the other hand WSO2 is fixing the bugs quickly but of course every bug is still an annoyance. So I have mixed feelings here.

Don't let the previous sentence fool you. IMO Ballerina is an excellent tool for a code that communicates over network with other services. It's also a very decent general purpose programming language with excellent tooling and modern language features.
