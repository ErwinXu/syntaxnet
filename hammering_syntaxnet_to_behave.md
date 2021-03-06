
## Hammering SyntaxNet to behave:
This blog is a listing of errors encountered while setting it up and how I hammered them to make it behave.
<br/>
**__Note:__** I've also blogged the [final working installation instructions to get syntaxnet import working in python](https://github.com/spoddutur/syntaxnet/edit/master/syntaxnet_setup_for_osx.md).
	
## 1. Build Errors
Build is done with `bazel test --linkopt=-headerpad_max_install_names dragnn/... syntaxnet/... util/utf8/...` command.
I ran into a myriad of issues before getting the build successful with all tests passing.
Iam listing some of the issues I faced here and how i got them fixed and hoping to help some SyntaxNet enthusiast out there facing similar issues :)]

### 1.1. Current Bazel version is 0.11.1, expected at least 0.4.5
<hr/>

#### Issue:
```diff
- Current Bazel version is 0.11.1, expected at least 0.4.5
- ERROR: Error evaluating WORKSPACE file
- ERROR: error loading package 'external': Package 'external' contains errors
INFO: Elapsed time: 0.127s
- FAILED: Build did NOT complete successfully (0 packages loaded)
```
#### Cause: 
Initially, I had latest bazel version installed using `pip install bazel` (0.11.1).
#### Fix: 
Downgraded it to 0.5.1 as mentioned in the error.

### 1.2. Bazel 0.5.1 install failed: 
<hr/>

Installing bazel with `brew install bazel@0.5.1` threw **Can't find bundle for base name com.google.errorprone.errors, locale en_IN** error as shown below
#### Issue:
```diff
- Last 15 lines from /Users/surthi/Library/Logs/Homebrew/bazel@0.5.1/01.compile.sh:
- at com.google.devtools.build.buildjar.BazelJavaBuilder.processRequest(BazelJavaBuilder.java:89)
- at com.google.devtools.build.buildjar.BazelJavaBuilder.runPersistentWorker(BazelJavaBuilder.java:66)
- at com.google.devtools.build.buildjar.BazelJavaBuilder.main(BazelJavaBuilder.java:44) Caused by: - - - java.util.MissingResourceException: Can't find bundle for base name com.google.errorprone.errors, locale en_IN
- at java.util.ResourceBundle.throwMissingResourceException(ResourceBundle.java:1573)
- at java.util.ResourceBundle.getBundleImpl(ResourceBundle.java:1396)
- at java.util.ResourceBundle.getBundle(ResourceBundle.java:854)
- at com.sun.tools.javac.util.JavacMessages.lambda$add$0(JavacMessages.java:106)
```
#### Cause: 
Upon investigating this, seemed like this is a common error that people posted online with bazel installation via brew.
#### Fix: 
Downloaded installer from bazel release version and installed.
```markdown
Download bazel-0.5.4-without-jdk-installer-darwin-x86_64.sh from  https://github.com/bazelbuild/bazel/releases 
chmod +x ~/Downloads/bazel-0.5.4-without-jdk-installer-darwin-x86_64.sh
sh ~/Downloads/bazel-0.5.4-without-jdk-installer-darwin-x86_64.sh
```

### 1.3 py_proto_library py_libs += [default_runtime] trying to mutate a frozen object
<hr/>

### Issue:
```diff
- “.../syntaxnet/syntaxnet/syntaxnet.bzl”, line 53, in tf_proto_library_py py_proto_library(name = name, srcs = srcs, srcs_versi...", <5 more arguments>) File 
- "/private/var/tmp/_bazel_XXX/f74e5a21c3ad09aeb110d9de15110035/external/protobuf_archive/protobuf.bzl", line 374, in py_proto_library py_libs += [default_runtime] trying to mutate a frozen object 
- ERROR: package contains errors: dragnn/protos
```
### Cause: 
This was a bug in bazel. It was fixed in later versions.
### Fix: 
Install 0.5.4 version of bazel. (This version has the fix for that)

### 1.4 Failing tests in bazel build
<hr/>

### 1.4.1 Issue: 
Upon introspection, main error source is dependency on autograd python package
```markdown
cat /root/.cache/bazel/_bazel_root/3b4c7ccb85580bc382ce4a52e9580003/execroot/__main__/bazel-out/local-opt/testlogs/syntaxnet/util/resources_test/test.log
from autograd import core as ag_core ImportError: No module named autograd
```
### Fix: 
pip install autograd

### 1.4.2 Issue: 
Cannot import name container_types
### Cause: 
Upon fixing with pip install autograd succesfully installs the package and throws a name import error
```diff
- cat /root/.cache/bazel/_bazel_root/3b4c7ccb85580bc382ce4a52e9580003/execroot/__main__/bazel-out/local-opt/testlogs/syntaxnet/util/resources_test/test.log
- from autograd import container_types ImportError: cannot import name container_types
```
### Fix: 
We need to install a compatible version of autograd (1.1.13)
`pip install autograd==1.1.13`. Also, again, make sure you have Bazel 0.5.4

### 1.4.3. Issue:
```diff
- /private/var/tmp/_bazel_surthi/81559a7957a70ca9917043de0cb80034/execroot/__main__/bazel-out/local-opt/testlogs/dragnn/python/graph_builder_test/test.log
- //dragnn/python:render_parse_tree_graphviz_test                          FAILED in 7.2s
- /private/var/tmp/_bazel_surthi/81559a7957a70ca9917043de0cb80034/execroot/__main__/bazel-out/local-opt/testlogs/dragnn/python/render_parse_tree_graphviz_test/test.log
- //dragnn/python:render_spec_with_graphviz_test                           FAILED in 7.3s
- /private/var/tmp/_bazel_surthi/81559a7957a70ca9917043de0cb80034/execroot/__main__/bazel-out/local-opt/testlogs/dragnn/python/render_spec_with_graphviz_test/test.log
- //dragnn/python:visualization_test                                       FAILED in 2.1s
- /private/var/tmp/_bazel_surthi/81559a7957a70ca9917043de0cb80034/execroot/__main__/bazel-out/local-opt/testlogs/dragnn/python/visualization_test/test.log
- //examples/dragnn:test_run_all_tutorials                                 FAILED in 7.9s
```
#### Cause: 
Bazel build failed due to two things. One, protobuf version incompatibility and two, missed installing pygraphviz.
#### Fix:
```markdown
pip install -U protobuf==3.2.0
pip install pygraphviz
```
#### With those, the bazel test passed with no failing tests. But still there were some more hurdles in getting the wrapper as mentioned in the next section.

## 2. Errors Post Successful Installation of Syntaxnet
Now, open python shell and try importing syntaxnet. Couple of classes that I need were not available. This could be because of the restructuring done as part of [The Great Models Move change](https://github.com/tensorflow/models/pull/2430). Let's look at some of the failing imports I needed and how i got them fixed.

### 2.1 "from syntaxnet.ops import gen_parser_ops" import FAILED
#### Issue: 
gen_parser_ops tries to import 'eager' module from tensorflow which resulted in import error.
```diff
- from tensorflow.python.eager import execute as _execute
- ImportError: No module named eager
```
#### Cause: 
tensorflow version i had wasnt compatible with latest syntaxnet - lot of restructuring happened.

#### Fix:
Add `--include-tensorflow` while building syntaxnet with bazel. This will bundle syntaxnet with tensorflow that is compatible with it.
```markdown
bazel-bin/dragnn/tools/build_pip_package --output-dir=/tmp/syntaxnet_pkg **-—include-tensorflow**
sudo pip install /tmp/syntaxnet-pkg/syntaxnet_with_tensorflow-0.2-cp27-cp27m-macosx_10_6_intel.whl
```
### 2.1 "from syntaxnet import graph_builder" and "from syntaxnet import structured_graph_builder" imports FAILED
#### Issue:
Syntaxnet bundle didnt include these two files.
#### Fix:
Manually edited build_pip_package_runfiles to include these classes and did rebuild again.

## 3. Appendix
Couple of other links that helped me in resolving the issues I ran into:
- https://stackoverflow.com/questions/47688252/tensorflow-trying-to-mutate-a-frozen-object-bazel
- https://github.com/tensorflow/models/issues/1271
- https://stackoverflow.com/questions/45991520/tensorflow-image-retraining-tutorial-bazel-error
- https://github.com/tensorflow/models/issues/2355
- https://github.com/tensorflow/models/issues/1271
- https://github.com/bazelbuild/bazel/issues/4483
- https://github.com/tensorflow/tensorflow/issues/16662
- https://github.com/bazelbuild/bazel/issues/3018
- https://www.tensorflow.org/install/install_mac
