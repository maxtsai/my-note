### JDK 1.6.0_45

<pre>
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update && sudo apt-get install oracle-java6-installer
java -version
</pre>

<pre>
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
sudo update-alternatives --config jar
sudo update-alternatives --config javadoc
sudo update-alternatives --config javap
</pre>

Or change java setting by update-java-alternatives

<pre>
sudo update-java-alternatives -l
sudo update-java-alternatives -s java-1.7.0-openjdk-amd64
</pre>
