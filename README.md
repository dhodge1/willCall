willCall
========


Please afford me the great pleasure of introducing willCall – a music/concert discovery tool built to work off of the last.fm social music site. Bear with me as I elaborate on the details of the cogs and widgets that drive my creation…shall we begin? Let’s.

 The Basic Idea


Since this is my first real event-driven program I didn’t really know how the flowchart should be done so my apologies if I am off-base somewhere in formatting. Essentially, the user supplies a last.fm username gets the top five artists for that user and user play counts for said artists then sets these as text labels on the next screen. In order to use the information provided by last.fm some parsing was needed (more on that below) to slice-and-dice the appropriate bits of data out of a great big mess. After the top five artists and play counts have been set the user has the option to either get concert information or see top tracks for each artist. When either of those buttons are clicked the program fetches the data and displays it in it’s appropriate text area. If at any point the quit button is clicked then the program does exactly that…quits.

The Nitty-Gritty Details

So before I dive into the guts of willCall specifically I would like to talk about  the API used to build this application. The last.fm java bindings is a java wrapper for the last.fm API. However, this is not a first party affair, but rather was developed by a third party and as such documentation is fairly limited so at times I felt a little like a blind man without a cane. However, the bindings provide classes and methods for almost everything supported by the official API so after a little trial-and-error things started to come together. The main classes  that I utilized were User, Events, Tracks & Venue. As I discuss these classes I’ll add dialog pertaining to what kind of data the methods store/present,  how I utilized the classes in my code and the modifications I had to come up with to make things work.

User & Artist



 After setting the API key and having the user search for a specified username, the above screen populates showing the top five artists, their respective play counts by the specified username and buttons to see concert and track info.  The text labels and play count labels for the top five artists are variable based upon username supplied and come from the getTopArtists() method in the User class. The method returns a Collection<Artist> and calls for a username string and api key string as parameters. But what the heck is a collection of artists? I thought the same thing. Well a collection represents a group of objects and Artist is basically a type of object created by Jonni Kovacs when he made this wrapper. Collections serve as an entire java framework for things like lists, queues and maps. You can learn more about what they are and how to use them here. So getTopArtists() returned a collection of artist objects for the specified last.fm username which looks a little something like this in the raw:

Artist[name='Tobacco', id='null', url='www.last.fm/music/Tobacco', mbid='357d27b6-f7c8-4d8b-84f2-8708678fb626', playcount=-1, listeners=-1, streamable=true]Artist[name='The Octopus Project & Black Moth Super Rainbow', id='null', url='www.last.fm/music/The+Octopus+Project+&+Black+Moth+Super+Rainbow', mbid='', playcount=-1, listeners=-1, streamable=false]Artist[name='The Seven Fields of Aphelion', id='null', url='www.last.fm/music/The+Seven+Fields+of+Aphelion', mbid='583f633c-08e1-4986-a9f5-d5cedeee28f2', playcount=-1, listeners=-1, streamable=true]Artist[name='The Octopus Project', id='null', url='www.last.fm/music/The+Octopus+Project',mbid='aaf92b5a-199c-4b1b-969d-7a915ffb6f0f', playcount=-1, listeners=-1, streamable=true]Artist[name='satanstompingcaterpillars', id='null',url='www.last.fm/music/satanstompingcaterpillars', mbid='8d23262a-67b2-4115-80f3-2bab4862d77f', playcount=-1, listeners=-1, streamable=false]Artist[name='Dreamend', id='null', url='www.last.fm/music/Dreamend', mbid='e77fc1ee-5bfb-4f16-91c0-fdd161e65701', playcount=-1, listeners=-1, streamable=true]Artist[name='Animal Collective', id='null', url='www.last.fm/music/Animal+Collective', mbid='0c751690-c784-4a4f-b1e4-c1de27d47581', playcount=-1, listeners=-1, streamable=true]Artist[name='Power Pill Fist', id='null', url='www.last.fm/music/Power+Pill+Fist', mbid='27fabbb8-90bc-4e12-bce0-6ac26f397aee', playcount=-1, listeners=-1, streamable=true]
 

Fun right. Well I couldn’t really use that for a few reasons…there is way too much garbage info in there that I don’t need, it’s not string data (it’s a special type of object remember!) so I couldn’t use it as content for the text labels and it’s not ordered in any real way. So I had to do a few things to get this to work. First I needed to get this data out of it’s collection and into something I knew how to deal with like an array. Lucky me I could make an Artist array the same size as the collection and copy the contents into the array. But that’s still not good enough, now instead of an object collection I’ve got an object array. I need strings! String array to the rescue – copy the object array into a string array of the same size. Yay! Now I’ve got some data I can use.  But wait…it’s still all jumbled up and I only need the artist name and playcount. Luckily each artist object was saved into a separate element in the string array. Time to split() and trim()! So now I need to take the initial string array and split it into another string array split on commas.  So the zero-th element of this new string array now looks like:

Artist[name=' Tobacco'

Ok...all I need is the string Tobacco so lets split this string array into another one split by apostrophes. That's it! Now the 1st element of this string array is what we need:

Tobacco

Lets store that in yet another string array to call upon later for our text labels and let's put  all of this in a for loop so we don't have to do it for every artist in the collection. I also needed the play count for each artist so I did a similar process but for the second split I used "=" rather than apostrophes to get the number that I needed. Now all we need to do is set the elements of our new string arrays to appear as the text labels for the top five artists and their respective play counts. Below is the code for this entire section, please forgive the silly array names I did this very late at night:

private void enterButtonActionPerformed(java.awt.event.ActionEvent evt) {
// TODO add your handling code here:
user = usernameTextField.getText();
topArtists = User.getTopArtists(user, key);
artists = topArtists.iterator();
Artist[] lol3 = new Artist[topArtists.size()];
lol3 = topArtists.toArray(lol3);
String[] derp = new String[lol3.length];
for(int q = 0; q<lol3.length; q++){
derp[q] = lol3[q].toString();
}
//String[] arName = new String[5];
for(int b = 0; b < 5; b++){
//System.out.println(derp[b]);
String[] test = derp[b].split(“, “);
//System.out.println(test[0].trim());
String[] test2 = test[0].split(“‘”);
//System.out.println(test2[1].trim());
arName[b] = test2[1].trim();
}
//String[] plc = new String[5];
for(int d = 0; d < 5; d++){
//System.out.println(derp[d]);
String[] jest = derp[d].split(“, “);
//System.out.println(jest[0].trim());
String[] jest3 = jest[4].split(“=”);
//System.out.println(jest3[1].trim());
plc[d] = jest3[1].trim();
}
band1Label.setText(arName[0]);
pc1Label.setText(plc[0]);
band2Label.setText(arName[1]);
pc2Label.setText(plc[1]);
band3Label.setText(arName[2]);
pc3Label.setText(plc[2]);
band4Label.setText(arName[3]);
pc4Label.setText(plc[3]);
band5Label.setText(arName[4]);
pc5Label.setText(plc[4]);
}
Event & Venue 



In order to get the concert data for specific artists I utilized the getEvents() method which returns a PaginatedResult<Event> and requires a string for the artist name and the API key…huh, paginated who? So remember all of that extra junk I had to do because of the fact that collections are kind of weird? Well apparently so does the creator of the wrapper because he saw fit to create another custom class called PaginatedResult. A paginated result is like a collection except that it is dynamic in that it anticipates that the amount of data returned might be very large so it paginates them and stores the results on an appropriate number of pages. Doesn’t sound like it’s any better does it…it actually sounds even more confusing. Well about 24 hours straight of cussing and crying led me to the fact that Jonni actually made it much easier and included methods in the Venue class to access and interpret the metadata within the paginated results as strings. Score.  So instead of doing all that junk with parsing, I could just access the data I needed directly and in a format that I wanted by creating a new Venue object (a variable from the event string array iterated through each element) and by using the following methods:

getName()
getStreet()
getCity()
getCountry()
getPostal()
getPhonenumber()
getDate()
With these super handy methods now I could get the venue name, address, date and contact information for all the upcoming concerts of an artist! Finally, concatenated the separate strings into one giant string and store each one of these as a different element in the array. Then I cycled through each string element and appended them as new rows in the concert text area.

The code is as follows:

private void event1ButtonActionPerformed(java.awt.event.ActionEvent evt) {
// TODO add your handling code here:
concertTextArea.setText(“”);
eventForArtist = Artist.getEvents(arName[0], key);
collEvent = eventForArtist.getPageResults();
events = collEvent.iterator();
de.umass.lastfm.Event[] lol4 = new de.umass.lastfm.Event[collEvent.size()];
lol4 = collEvent.toArray(lol4);
String[] arCons = new String[lol4.length];
for(int l = 0; l<lol4.length; l++) {
Venue ven = lol4[l].getVenue();
String vName = ven.getName();
String vAddr = ven.getStreet();
String vCity = ven.getCity();
String vCountry = ven.getCountry();
String vPost = ven.getPostal();
String vPhone = ven.getPhonenumber();
String vInfo = vName + “, ” + vAddr + “, ” + vCity + “, ” + vCountry + “, ” + vPost + “, ” + vPhone;
arCons[l] = vInfo;
}
for(int x = 0; x < arCons.length; x++) {
concertTextArea.append(arCons[x] + newline);
}
}
 

Track



The process for getting track data was very similar to getting top artist data in that initially I was presented with a collection of the top tracks for an artist then I had to parse out the track name, stick it into a string array, do that for each song and then append each element as a new row in the tracks text area. Here is the code:

private void track1ButtonActionPerformed(java.awt.event.ActionEvent evt) {
// TODO add your handling code here:
tracksTextArea.setText(“”);
topTracks = Artist.getTopTracks(arName[0], key);
tracks = topTracks.iterator();
Track[] lol2 = new Track[topTracks.size()];
lol2 = topTracks.toArray(lol2);
String[] trax = new String[lol2.length];
for(int z = 0; z < lol2.length; z++){
trax[z] = lol2[z].toString();
}
String[] yarg = new String[trax.length];
for(int u = 0; u<trax.length; u++){
String[] nest = trax[u].split(“,”);
String[] nest2 = nest[0].split(“=”);
yarg[u] = nest2[1].trim();
}
for(int o = 0; o < yarg.length; o++) {
tracksTextArea.append(yarg[o] + newline);
}
}
 

Member Variables

I probably should have listed this in the beginning but I didn’t…oh well here are the member variables and imported libraries that are used throughout the application:

import de.umass.lastfm.*;
import java.awt.*;
import java.awt.event.*;
import java.util.Collection;
import java.util.Iterator;
public class WillCall extends javax.swing.JFrame {
/**
* Creates new form WillCall
*/
String key = “7cc01cf3a6d18242a5d58ca02677b5ae”;
String user;
Collection<Artist> topArtists;
Iterator<Artist> artists;
String[] arName = new String[5];
String[] plc = new String[5];
PaginatedResult<de.umass.lastfm.Event> eventForArtist;
Collection<de.umass.lastfm.Event> collEvent;
Iterator<de.umass.lastfm.Event> events;
Collection<Track> topTracks;
Iterator<Track> tracks;
private final static String newline = “\n”;
 

Future

So that’s pretty much it as it stands right now. That wasn’t too bad now was it? Pretty simple code all in all, but it’s repeated a whole lot because each button requires very similar code with just a few variable sources changed here and there and some other minor differences that kept me from automating the whole process. In the future I would like to create methods that I can call to execute the data parsing and formatting. In this way I could eliminated much of the repeated code, just call the method and cut down on the number of lines required. I dunno, I’ll figure something out. I’d also like to make the application a lot more visual, instead of showing just text and numbers I could have a clickable graph that when clicked goes into an artist bio of some kind then allows the user to see upcoming concerts, provide the ability to purchase tickets (willCall…get it?) and show similar artists/upcoming concerts and top tracks for the similar artists to add to the music discovery aspects of the application. Maybe if I could put the artists on like a connected spider web type graph that would look pretty neat. Aside from the graphical aspects of what I just mentioned, the last.fm API provides methods for the rest of it so it shouldn’t be too difficult. Over Christmas break I’m going to port this over to Android as a free application to start familiarizing myself with mobile development. I’ll let you know how it goes…



