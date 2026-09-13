This project uses a multi-branch Jenkins pipeline to increment the version in pom.xml, build the app using maven clean and maven package, 
build and store the image in a private image repository on AWS Elastic Container Registry, then deploy the app on Elastic Kubernetes Service.
Finally, the app's updated version gets committed and pushed back up to GitHub. 

This demonstrates hands-on experience with a complete continuous integration, continuous delivery pipeline along with ancillary technologies
and concepts such as version control for code and docker images, build tools like Maven. 

Projects like this are valuable because there should always be an auditable record of what happened on a system and when, in case a failure or break requires a rollback to a known-good version, and because managing a process like this manually is beyond time-intensive, and at scale, automating the process becomes a necessity. 
