Instructions moved to https://github.com/skillstream/chef-kitchen/wiki/Big-Brother:-installing-the-monitoring-client

Scratch that, it didn’t work. The trick seems to be to use mksh instead: sudo yum install mksh and then change the top line of bb-roracle.ksh to 
`!/bin/mksh`
Simples.