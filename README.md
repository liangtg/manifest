A description of the elements and their attributes follows.
Element manifest
The root element of the file.
Element remote
One or more remote elements may be specified. Each remote element specifies a Git URL shared by one or more projects and (optionally) the Gerrit review server those projects upload changes through.
Attribute name: A short name unique to this manifest file. The name specified here is used as the remote name in each project's .git/config, and is therefore automatically available to commands likegit fetch,git remote,git pull and git push.
Attribute alias: The alias, if specified, is used to override name to be set as the remote name in each project's .git/config. Its value can be duplicated while attributename has to be unique in the manifest file. This helps each project to be able to have same remote name which actually points to different remote url.
Attribute fetch: The Git URL prefix for all projects which use this remote. Each project's name is appended to this prefix to form the actual URL used to clone the project.
Attribute review: Hostname of the Gerrit server where reviews are uploaded to byrepo upload. This attribute is optional; if not specified thenrepo upload will not function.
Element default
At most one default element may be specified. Its remote and revision attributes are used when a project element does not specify its own remote or revision attribute.
Attribute remote: Name of a previously defined remote element. Project elements lacking a remote attribute of their own will use this remote.
Attribute revision: Name of a Git branch (e.g. master orrefs/heads/master). Project elements lacking their own revision attribute will use this revision.
Element manifest-server
At most one manifest-server may be specified. The url attribute is used to specify the URL of a manifest server, which is an XML RPC service that will return a manifest in which each project is pegged to a known good revision for the current branch and target.
The manifest server should implement:
GetApprovedManifest(branch, target)
The target to use is defined by environment variables TARGETPRODUCT and TARGETBUILDVARIANT. These variables are used to create a string of the form $TARGETPRODUCT-$TARGETBUILDVARIANT, e.g. passion-userdebug. If one of those variables or both are not present, the program will call GetApprovedManifest without the target paramater and the manifest server should choose a reasonable default target.
Element project
One or more project elements may be specified. Each element describes a single Git repository to be cloned into the repo client workspace.
Attribute name: A unique name for this project. The project's name is appended onto its remote's fetch URL to generate the actual URL to configure the Git remote with. The URL gets formed as:
${remotefetch}/${projectname}.git
where ${remotefetch} is the remote's fetch attribute and ${projectname} is the project's name attribute. The suffix ".git" is always appended as repo assumes the upstream is a forrest of bare Git repositories.
The project name must match the name Gerrit knows, if Gerrit is being used for code reviews.
Attribute path: An optional path relative to the top directory of the repo client where the Git working directory for this project should be placed. If not supplied the project name is used.
Attribute remote: Name of a previously defined remote element. If not supplied the remote given by the default element is used.
Attribute revision: Name of the Git branch the manifest wants to track for this project. Names can be relative to refs/heads (e.g. just "master") or absolute (e.g. "refs/heads/master"). Tags and/or explicit SHA-1s should work in theory, but have not been extensively tested. If not supplied the revision given by the default element is used.
Attribute groups: List of groups to which this project belongs, whitespace or comma separated. All projects belong to the group "default", and each project automatically belongs to a group of it's name:name and path:path. E.g. for , that project definition is implicitly in the following manifest groups: default, name:monkeys, and path:barrel-of.
Element annotation
Zero or more annotation elements may be specified as children of a project element. Each element describes a name-value pair that will be exported into each project's environment during a 'forall' command, prefixed with REPO__. In addition, there is an optional attribute "keep" which accepts the case insensitive values "true" (default) or "false". This attribute determines whether or not the annotation will be kept when exported with the manifest subcommand.
Element remove-project
Deletes the named project from the internal manifest table, possibly allowing a subsequent project element in the same manifest file to replace the project with a different source.
This element is mostly useful in the local_manifest.xml, where the user can remove a project, and possibly replace it with their own definition.
Element include
This element provides the capability of including another manifest file into the originating manifest. Normal rules apply for the target manifest to include- it must be a usable manifest on it's own.
Attribute name; the manifest to include, specified relative to the manifest repositories root.
Local Manifest
Additional remotes and projects may be added through a local manifest, stored in$TOP_DIR/.repo/local_manifest.xml.
For example:
$ cat .repo/local_manifest.xml
Users may add projects to the local manifest prior to a repo sync invocation, instructing repo to automatically download and manage these extra projects.

1. manifest
这个是配置的顶层元素，即根标志
2. remote
name：在每一个.git/config文件的remote项中用到这个name，即表示每个git的远程服务器的名字(这个名字很关键，如果多个remote属性的话，default属性中需要指定default remote)。git pull、get fetch的时候会用到这个remote name。
alias ：可以覆盖之前定义的remote name，name必须是固定的，但是alias可以不同，可以用来指向不同的remote url
fetch ：所有git url真正路径的前缀，所有git 的project name加上这个前缀，就是git url的真正路径
review ：指定Gerrit的服务器名，用于repo upload操作。如果没有指定，则repo upload没有效果
3. default 
设定所有projects的默认属性值，如果在project元素里没有指定一个属性，则使用default元素的属性值。
remote ：远程服务器的名字（上面remote属性中提到过，多个remote的时候需要指定default remote，就是这里设置了）
revision ：所有git的默认branch，后面project没有特殊指出revision的话，就用这个branch
sync_j ： 在repo sync中默认并行的数目
sync_c ：如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容
sync_s ： 如果设置为true，则会同步git的子项目
4. manifest-server
它的url属性用于指定manifest服务的URL，通常是一个XML RPC 服务
它要支持一下RPC方法：
GetApprovedManifest(branch, target) ：返回一个manifest用于指示所有projects的分支和编译目标。
    target参数来自环境变量TARGET_PRODUCT和TARGET_BUILD_VARIANT，组成$TARGET_PRODUCT-$TARGET_BUILD_VARIANT
GetManifest(tag) ：返回指定tag的manifest
5. project
需要clone的单独git
name ：git 的名称，用于生成git url。URL格式是：${remote fetch}/${project name}.git 其中的 fetch就是上面提到的remote 中的fetch元素，name 就是此处的name
path ：clone到本地的git的工作目录，如果没有配置的话，跟name一样
remote ：定义remote name，如果没有定义的话就用default中定义的remote name
revision ：指定需要获取的git提交点，可以定义成固定的branch，或者是明确的commit 哈希值
groups ：列出project所属的组，以空格或者逗号分隔多个组名。所有的project都自动属于"all"组。每一个project自动属于
name:'name' 和path:'path'组。例如<project name="monkeys" path="barrel-of"/>，它自动属于default, name:monkeys, and path:barrel-of组。如果一个project属于notdefault组，则，repo sync时不会下载
sync_c ：如果设置为true，则只同步指定的分支(revision 属性指定)，而不是所有的ref内容。
sync_s ： 如果设置为true，则会同步git的子项目
upstream ：在哪个git分支可以找到一个SHA1。用于同步revision锁定的manifest(-c 模式)。该模式可以避免同步整个ref空间
annotation ：可以有0个或多个annotation，格式是name-value，repo forall命令是会用来定义环境变量
6. include
通过name属性可以引入另外一个manifest文件(路径相对与当前的manifest.xml 的路径)
name ：另一个需要导入的manifest文件名字
