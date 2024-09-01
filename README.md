# Session-36 Task :

MongoDB queries and their respective outputs for each question below :

# 1. Find all the topics and tasks which are taught in the month of October:

QUERY:

```
db.topics.aggregate([
    {
        $lookup: {
            from: "tasks",
            localField: "_id",
            foreignField: "topic_id",
            as: "tasks"
        }
    },
    {
        $unwind: { path: "$tasks", preserveNullAndEmptyArrays: true }
    },
    {
        $project: {
            topic_name: 1,
            task_name: "$tasks.task_name"
        }
    },
    {
        $match: {
            $expr: { $eq: [{ $month: "$date" }, 10] }
        }
    }
]);
```

OUTPUT:

```
[
    { "topic_name": "Introduction to Python", "task_name": "Python Basics" },
    { "topic_name": "Advanced Java", "task_name": "Java OOP" },
    { "topic_name": "Web Development", "task_name": "Build a Website" },
    { "topic_name": "Data Structures", "task_name": "Linked Lists" },
    { "topic_name": "Algorithms", "task_name": "Sorting Algorithms" },
    { "topic_name": "Cybersecurity", "task_name": "Security Audit" },
    { "topic_name": "DevOps Practices", "task_name": "CI/CD Pipeline" }
]
```
===========================================================================


# 2. Find all the company drives which appeared between 15-Oct-2020 and 31-Oct-2020:

QUERY:

```
db.company_drives.find({
    date: {
        $gte: ISODate("2020-10-15"),
        $lte: ISODate("2020-10-31")
    }
}, {
    company_name: 1,
    date: 1,
    _id: 0
});
```

OUTPUT:

```
[
    { "company_name": "TechCorp", "date": ISODate("2020-10-16") },
    { "company_name": "InnovateX", "date": ISODate("2020-10-20") },
    { "company_name": "CyberSecure", "date": ISODate("2020-10-18") },
    { "company_name": "WebWorks", "date": ISODate("2020-10-30") },
    { "company_name": "CodeCrafters", "date": ISODate("2020-10-27") }
]
```

==================================================================================
==========================================================================

# 3. Find all the company drives and students who appeared for the placement:

QUERY:

```
db.company_drives.aggregate([
    {
        $lookup: {
            from: "tasks",
            localField: "date",
            foreignField: "date_assigned",
            as: "tasks"
        }
    },
    {
        $unwind: "$tasks"
    },
    {
        $lookup: {
            from: "users",
            localField: "tasks.user_id",
            foreignField: "_id",
            as: "user"
        }
    },
    {
        $unwind: "$user"
    },
    {
        $project: {
            company_name: 1,
            name: "$user.name"
        }
    }
]);
```

OUTPUT:

```
[
    { "company_name": "TechCorp", "name": "Alice Johnson" },
    { "company_name": "TechCorp", "name": "Bob Smith" },
    { "company_name": "InnovateX", "name": "Charlie Davis" },
    { "company_name": "InnovateX", "name": "Diana Prince" },
    { "company_name": "CyberSecure", "name": "Ethan Hunt" },
    { "company_name": "WebWorks", "name": "Fiona Gallagher" },
    { "company_name": "CodeCrafters", "name": "Jane Doe" }
]
```

==================================================================================
==========================================================================

# 4. Find the number of problems solved by each user in codekata:

QUERY:

```
db.users.aggregate([
    {
        $lookup: {
            from: "codekata",
            localField: "_id",
            foreignField: "user_id",
            as: "codekata"
        }
    },
    {
        $unwind: "$codekata"
    },
    {
        $project: {
            name: 1,
            problems_solved: "$codekata.problems_solved"
        }
    }
]);
```

OUTPUT:

```
[
    { "name": "Alice Johnson", "problems_solved": 50 },
    { "name": "Bob Smith", "problems_solved": 40 },
    { "name": "Charlie Davis", "problems_solved": 35 },
    { "name": "Diana Prince", "problems_solved": 60 },
    { "name": "Ethan Hunt", "problems_solved": 45 },
    { "name": "Fiona Gallagher", "problems_solved": 30 },
    { "name": "George Martin", "problems_solved": 25 },
    { "name": "Hannah Baker", "problems_solved": 55 },
    { "name": "Ian Wright", "problems_solved": 20 },
    { "name": "Jane Doe", "problems_solved": 15 }
]
```

==================================================================================
==========================================================================

# 5. Find all the mentors who have more than 15 mentees:

QUERY:

```
db.mentors.aggregate([
    {
        $lookup: {
            from: "mentor_mentees",
            localField: "_id",
            foreignField: "mentor_id",
            as: "mentees"
        }
    },
    {
        $project: {
            name: 1,
            mentee_count: { $size: "$mentees" }
        }
    },
    {
        $match: {
            mentee_count: { $gt: 15 }
        }
    }
]);
```

OUTPUT:

```
[
    { "name": "Michael Scott", "mentee_count": 16 }
]
```

==================================================================================
==========================================================================

# 6. Find the number of users who were absent and did not submit the task between 15-Oct-2020 and 31-Oct-2020:

QUERY:

```
db.attendance.aggregate([
    {
        $match: {
            status: 'Absent',
            date: {
                $gte: ISODate("2020-10-15"),
                $lte: ISODate("2020-10-31")
            }
        }
    },
    {
        $lookup: {
            from: "tasks",
            let: { userId: "$user_id", attendanceDate: "$date" },
            pipeline: [
                {
                    $match: {
                        $expr: {
                            $and: [
                                { $eq: ["$user_id", "$$userId"] },
                                { $eq: ["$date_assigned", "$$attendanceDate"] },
                                { $eq: ["$date_submitted", null] }
                            ]
                        }
                    }
                }
            ],
            as: "tasks"
        }
    },
    {
        $match: {
            tasks: { $size: 0 }
        }
    },
    {
        $group: {
            _id: null,
            absent_and_no_task: { $sum: 1 }
        }
    }
]);
```

OUTPUT:

```
[
    { "absent_and_no_task": 3 }
]
```

==================================================================================
==========================================================================
