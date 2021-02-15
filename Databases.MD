### [HOME](https://laswiz.github.io/turbo-bassoon/index.html) [About Me](https://laswiz.github.io/turbo-bassoon/AboutMe.html) [Enhanced Chiller Module](https://laswiz.github.io/turbo-bassoon/ChillerControl.html) [File Handling](https://laswiz.github.io/turbo-bassoon/FileHandling.html) [Databases](https://laswiz.github.io/turbo-bassoon/Databases.html)

## Where Do I Start?
Being a PLC developer, you rarely get the chance to work in databases. I have certainly done so in my career, but normally it is just cleaning up other peoples’ messes, 
or just making sense of some data for some project. Of all the databases I have worked on (and there have not been that many), none rose to the level of portfolio material. 
So instead, I decided to focus on one of the classes I have taken on this path to my bachelor’s degree at SNHU. IT-315 is focusing on something I want to get better at, 
object-oriented design. In that class we were required to design a database access for a fictitious school with a couple of criteria: students and staff must both access it, 
and must be restricted to the information they are allowed to access. For the project, we must only show the hierarchy of the system in general, no code was necessary. I did 
well in this assignment, but there were several things that were missing. What I wanted to focus on for this artifact is polishing up the SIS class diagram that I submitted 
with this class:

![Picture 6](https://laswiz.github.io/turbo-bassoon/assets/Picture6.png)

## OH, THAT'S How You Do It!
I decided the best way to promote how I approach a problem is by refining this plan further. Thinking of how a user would want to interact with the system, and especially try 
and exploit the system (after my secure coding class, I just can’t help thinking the worst now), I wanted to protect the data further. I removed the association between the
student and the maintenance and instead created a basic class that will handle only those functions the student is allowed to access. I also added a user level to both 
students and staff, but the students will have a fixed “basic” user level. This will provide the best protection that I can think of in a database while still allowing the 
student and staff to use the same database. Further enhancements I could think of are using lookup tables for the student activities. In this way, the data can be fed as 
read only to the student information, and no changes would be allowed from the student perspective. This would further enhance and protect the database by limiting the amount 
of users who can access the writable database itself. One of the things that really frustrates me with people and databases is misusing them. I worked for a company which had 
a monolithic approach to databases; that is, one table with 255 columns. The big problem was that many were duplicated, but used for different machine types. Rather than 
combining the data, or putting like data into different tables, they just kept adding columns. Eventually, they had to add another table, and then another, because this 
approach grew exponentially. I have learned from this and like lookup tables and grouping like data together. In this case, the classes themselves can have a table. Even 
though we have a couple of entries in each that are not needed, it will still be more efficient to store it this way. And who knows? Maybe there will be a class offered 
virtually AND online at the same time! I am very hopeful that I have met the requirements. The other two artifacts were very easy to do, but this one I do not have a lot of 
data to pull from. The other challenge I faced is that I no longer had the program available to edit the file, as I took that class quite a while ago. However I was able to
procure an evaluation copy so I could make the necessary changes to show my enhancement. I would love to bring this to fruition; ok, not this exact product, but something 
similar. I think it could be used in any number of places from the workplace to volunteer organizations. Then again, many of these are already implemented, so the actual task
would produce more benefits than the final product. In the end, below is my updated database design for a student identification system:

![Picture 7](https://laswiz.github.io/turbo-bassoon/assets/Picture7.png)
