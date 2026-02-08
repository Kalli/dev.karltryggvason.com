+++
date = '2026-02-08T14:16:04+01:00'
title = 'BFF - Semver mnemonic'
+++

Recently I've been parsing a bunch of open source software release data. Semantic versioning seemed to be the most common versioning schema, it is encouraged by many package managers and a de facto standard in the space. Here is the official summary, in the unlikely case that you are not familiar with it:

> Given a version number MAJOR.MINOR.PATCH, increment the:   
> 1. MAJOR version when you make incompatible API changes   
> 2. MINOR version when you add functionality in a backward compatible manner   
> 3. PATCH version when you make backward compatible bug fixes
> 
> Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.
>
> -- <cite>[Semantic Versioning 2.0.0](https://semver.org)</cite>

While doing this parsing work, I came across [this interesting issue in the semver repo](https://github.com/semver/semver/issues/625): "*The terms MAJOR.MINOR.PATCH are not semantic*". Quoting the issue in its entirety: 

> _Problem_   
> Semver uses the historically significant terms `MAJOR.MINOR.PATCH`. While these terms are intuitive, they are not semantic. I have noticed that maintainers think (incorrectly?) that the terms relate to the amount of change to the code or the API which is inconsistent with semver. For instance, a maintainer may refactor their entire library, but not change the public API in anyway. According to semver, this would be a PATCH change, even though the entire library was rewritten. This introduces a lot of unnecssary [sic] confusion.
>
> Proposed Solution   
> Rename MAJOR.MINOR.PATCH to BREAKING.FEATURE.FIX which is a more semantic represenation [sic] of the version string.

I've definitely noted the same sort of confusion when working with other developers, equating the `MAJOR` and `MINOR` names with the amount of work behind the release, feeling that a `MINOR` release does injustice to a lot of hard work. Beyond the psychological sting of a `MINOR` tag I think the `BREAKING` bit also communicates _breaking_ changes better to any consumers or dependents of the new release. 

A change of the naming in semver seems unlikely to happen at this point (the issue has been sitting there since 2020). But I do like the alternate breakdown of breaking/feature/fix plus it has a nice mnemonic in BFF. That's why the _BFF_ will be my new way to think about and communicate around semantic versioning. The next time someone on your team wants to bump `MAJOR` because a lot of hard work went into a release, ask them about BFF: _Breaking_, _Feature_, or _Fix_?
