# lnut-snippets
Some short snippets of JS that I wrote for languagenut back in the day.

notes:
- these are super hacky
- i don't know if they still work, if they have minified their JS source or webpacked it properly, these are highly likely to break since they depend on actual variable names
- some of these will not play nicely with a lot of the website, as they rely heavily on monkeypatching, so many things may be broken.
- however, there are some which i think will be impossible to patch out fully.

I've also tried to write one which fetches the answers and then sends them straight back to the server, but it ended up getting
pretty lengthy and hard to maintain. In the end, I decided that sort of functionality would be reserved for lnut-client, which 
is dependent on the same API rather than the same JS code.

## scripts
### script 1
```
ProgressManager.prototype.setGameData=TestProgressManager.prototype.setGameData=function(x){this.gameData=x;thing=this;}
ProgressManager.prototype.oldbuild=ProgressManager.prototype.build;TestProgressManager.prototype.oldbuild=TestProgressManager.prototype.build;
ProgressManager.prototype.build=()=>{thing.oldbuild();y()};TestProgressManager.prototype.build=()=>{thing.oldbuild();x()}
magic=()=>{return thing.setAnswer(true, thing.gameData.gameType=="writing"?thing.currentQuestionVocab.learningWord:thing.currentQuestionVocab)}
x=()=>{if(thing.currentQuestion < thing.catalog.length)magic().then(x)}
y=()=>{for(let i=0;i<thing.catalog.length;i++)magic()}
```
Monkeypatches the ProgressManager and TestProgressManager classes. Highly dependent on languagenut not name-mangling their classes, and also heavily
dependent on the actual implementation of the class.
- upon instantiation, it calls the original build function
- additionally it calls some custom code
- this calls the original `setAnswer()` method, which marks each question as complete and correct.
- then using its own logic, it sends the data off to the server

### script 2
```
m=async()=>{return q.setAnswer(1,q.currentQuestionVocab)};x=()=>{if(i++<q.catalog.length)m().then(x)};s=(c)=>{return `${c}setGameData=function(_){this.gameData=_;q=this;};${c}o=${c}build;${c}build=()=>{i=0;q.o();x()}`};p="ProgressManager.prototype.";eval(s(p)+";"+s("Test"+p))
```
Same as above, just minified quite a lot. Again, it depends on languagenut's own logic.

### script 3

```Util.seedRandomGenerator=function(seed){randomGenerator= ()=>{return 0}};s = setTimeout; setTimeout = function(a, b, ...x){s(()=>{try{a()}catch{}}, b <= 3000?1:b, ...x)}```
First replaces `setTimeout` with a version which ignores any delay less than 3 seconds,
then patches out the random number generator with one that returns a constant value, bypassing all of the scrambling (all answers will be in the same place)

I think this one will be very very difficult to patch out fully. as setTimeout is a standard browser thing. There are ways to defend, but
then it becomes a race to see who can load their version of setTimeout fastest. The RNG bypass could be patched out via minification, but then functions lower in the chain could be attacked. Again, this would eventually
just become a race to see who loads their RNG function first.

The method behind this script is usable in a very wide range of places - the setTimeout hack is pretty much universally portable, and
by switching `Math.random` or similar functions for functions like `()=>{some arbitrary constant]` random elements can be bypassed on a wide range of sites.
