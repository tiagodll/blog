---
title: "PetaPoco. Said whaaaat?"
date: 2012-03-24T17:31:40+02:00
draft: false
tags: ['csharp', 'dotnet']
---

So, maybe some of you guys checked out my dead database project, the dNet.Db. It is free and open source @ [http://dnetdb.codeplex.com/](http://dnetdb.codeplex.com/)
I started developing that framework because I always thought  it is important to automatize the simple and boring part of database. And by this I mean that it load an object values should be as easy as object.load(), or object.LoadList() if you want an array.
And it was working fine. But we all know how a project like that requires a lot of maintenance, adding new databases support, fixing bugs and things like that.

A few weeks ago I was talking about this to a friend and he told me about the project massive.
Massive is nice, but it doesn't support everything I needed (mainly because of the stored procedures), so trying to go further into that, I found the PetaPoco(http://www.toptensoftware.com/petapoco/).

This is the kind of framework I consider that fit all my needs:
- really easy to install
- easy to fetch objects from the db
- supports sqlite
- allows you to use plain old SQL for all the other situations.

So I've been using it for a bit now, and it hasn't let me down. I am not thinking about using my framework anymore. So I decided to write a little tutorial of how it works.

Install: There is a nuget for that. If you don't know what a nuget is, here is a link, probably I'll talk about this someday, but this is really cool, check it out.

Model:

{{< highlight csharp >}}
[PetaPoco.PrimaryKey("Id")]
public class Project
{
      public int Id { get; set; }
      public string Name { get; set; }
}
{{< / highlight >}}

Voil'a... Easy as pie...
then you could say:-Hum... the loading part might be tricky....
not really... this is the code:

var db = new PetaPoco.Database("Base");
ViewBag.Projects = db.Fetch<Models.Project>("");

and that's it.
Ok, that's not it. Well it is if you have a really simple object, but you don't.
So... now comes the tricky part?
Yes. Compared to the other part it is. but not so much

The loading bit is exactly the same, but now I'll add some more code to the model:
{{< highlight csharp >}}
   [PetaPoco.PrimaryKey("Id")]
    public class Feature
    {
        public int Id { get; set; }
        public int ParentId { get; set; }
        public int ProjectId { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        [PetaPoco.ResultColumn]
        public Project project { get; set; }
    }
    [PetaPoco.PrimaryKey("Id")]
    public class Project
    {
        public int Id { get; set; }
        public string Name { get; set; }
        [PetaPoco.ResultColumn]
        public Dictionary<long, Feature> Features { get; set; }
    }
    class ProjectFeatureRelator
    {
        Dictionary<long, Feature> features = new Dictionary<long, Feature>();

        public Project MapIt(Project p, Feature f)
        {
            Feature fExisting;
            if (features.TryGetValue(f.Id, out fExisting))
                f = fExisting;
            else
                features.Add(f.Id, f);

            f.project = p;
            p.Features = features;
            return p;
        }
    }
{{< / highlight >}}

Now, this looks a lot, but let's understand what happened there:
1 - I've added a new class (feature). This class have a primary key Id, and I want it to load the project associated with it.
2 - I've added a new property to the project class:Features. this is because I want the list of features to be loaded as well. Note that I've annotated the [PetaPoco.ResultColumn] to make the connection.
3 - So far so good, but now comes the ugly part: the join. I admit it is not an elegant code as the rest, but it is not so bad, and it is very easy, you just have to copy it, and use it in your code.

I don't know if it is a good idea to use this in a big project, but for the small ones, I will definitely use it. 