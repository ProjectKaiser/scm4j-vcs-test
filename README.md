# Overview
Pk-vcs-test project provides base functional test class for IVCS implementations subclassed from [pk-vcs-api](https://github.com/ProjectKaiser/pk-vcs-api). It used as maven dependency for [pk-vcs-git](https://github.com/ProjectKaiser/pk-vcs-git), [pk-vcs-svn](https://github.com/ProjectKaiser/pk-vcs-svn) and other VCS support libraries. All neccessary vcs-related functional testing is implemented within pk-vcs-test. It is need to implement just few abstract methods.

# Terms

- Abstract Test
  - Functional test of common behaviour of a certain VCS implementation. 
  - Exposed as VCSAbstractTest class
- Workspace Home
  - Home folder of all folders used by vcs-related operations. See [pk-vcs-api](https://github.com/ProjectKaiser/pk-vcs-api) for details
- Repository Workspace
  - Folder for LWC folders related to Repository of one type. See [pk-vcs-api](https://github.com/ProjectKaiser/pk-vcs-api) for details
- Locked Working Copy, LWC
  - Folder where vcs-related operations are executed. Provides thread- and process-safe repository of working folders. See [pk-vcs-api](https://github.com/ProjectKaiser/pk-vcs-api) for details
- Test Repository
  - A VCS Repository which is used to execute vcs operations which are being tested.
  - Generates new before and deletes after each test
  - Named randomly (uuid is used)

# Overall testing process

- Workspace Home folder is defined as `System.getProperty("java.io.tmpdir") + "pk-vcs-workspaces"`
- A new Test Repository name and Repository Workspace generates for each test within `setUp()`. Test Repository is named as `"pk-vcs-" + getVCSTypeString() + "-testrepo_" + uuid`
- A test method executes
- Mocks are verified. LWC closing is checked if one was obtained
- Workspace Home folder deletes. So all files and dirs within Workspace Home must be released

# Implementing VCS test
- Add github-hosted pk-vcs-test project as maven dependency using [jitpack.io](https://jitpack.io/). As a gradle exmaple, add following to gradle.build file:
```gradle
allprojects {
	repositories {
		maven { url "https://jitpack.io" }
	}
}

dependencies {
	testCompile 'com.github.ProjectKaiser:pk-vcs-test:master-SNAPSHOT'
}
```
- Create VCSAbstractTest subclass within test package
  - Override `setUp()` method
    - Create all neccessary data, test repositories and so on
    - Note that `vcs` instance is already created within `super.setUp()`
  - Create @After method if neccessary
  - Use `mockedVCSRepo` as `IVCSRepositoryWorkspace` parameter passed to VCS constructor. VCS implementation must use this `IVCSRepositoryWorkspace` for obtaining LWC. Also this instance will be tested for method calling using Mockito partial mocking by VCSAbstractTest
  - `getVCS(...)` method must return IVCS implementation which will be tested. Also this IVCS implementation must use provided `mockedVCSRepo`
      ```java
      @Override
    	protected IVCS getVCS(IVCSRepositoryWorkspace mockedVCSRepo) {
    		return new GitVCS(mockedVCSRepo);
    	}
      ```
- `getVCSTypeString()` method must return short VCS name, e.g. "git", "svn"
- `getTestRepoUrl()` method must return string url to Test Repository
- `getBranches()` method must return set of branches names from Test Repository
- `getCommitMessagesRemote(String branchName)` must return set of commit messages of branch `branchName` from Test Repository
- `checkout(String branchName, IVCSLockedWorkingCopy wc)` method must checout branch `branchName` to provided LWC from Test Repository
- `setMakeFailureOnVCSReset(Boolean doMakeFailure)` must make so next `merge` operation will fail on LWC reset caused by merge conflict. This need to test LWC corruption.
- `sendFile(...)` methods must commit an existing provided file and send it to test Repository. I.e. commit and push for Git, just commit for SVN.
- Use `localVCSWorkspace` field as Workspace Home
- Use `localVCSRepo` field for creating utility Locked Working Copies, e.g. for test content generating. See `getCommitMessagesRemote()` method in [pk-vcs-git](https://github.com/ProjectKaiser/pk-vcs-git)
- `mockedLWC` returns each time as a result of `mockedVCSRepo.getLockedWoringCopy()` call. If neccessary it could be used for additional testing. See `setMakeFailureOnVCSReset()` in [pk-vcs-git](https://github.com/ProjectKaiser/pk-vcs-git)
- Use `repoName` field to get current testing repository name. It generates again for each test
- Use `repoUrl` field to get url to current testing repository.
- Use `vcs` field as current IVCS implementation which is being testing

# Examples

- [pk-vcs-git](https://github.com/ProjectKaiser/pk-vcs-git)
- [pk-vcs-svn](https://github.com/ProjectKaiser/pk-vcs-svn)

# See also

- [pk-vcs-api](https://github.com/ProjectKaiser/pk-vcs-api)
