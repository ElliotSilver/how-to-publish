Many implementation guide (IG) authors are familiar with using [HL7](http://hl7.org)'s [IG Publisher](https://github.com/HL7/fhir-ig-publisher) to author and build an implementation guide. However the process of actually "publishing" an authored IG is less familiar to many. To make an IG available to the public (or at least to a larger audience than the author), it should be on an accessible website. Further, various FHIR tooling depends being able to find an IG and its packages listed in the central registries. This IG documents the process of publishing an IG to a website using IG Publisher. It also discusses the differences between "publishing" a FHIR IG by simply uploading the IG build output to your web server; and, by uploading the results of the IG Publisher `-go-publish` functionality.

This IG is not a tutorial on IG authoring; on general IG Publisher usage; or, on related tooling, such as [SUSHI](https://fshschool.org).

Once an IG has been authored using IG Publisher, there are several ways to make it available to the public:

* Using the HL7 [Auto-build](#ci-auto-build) functionality to publish the IG.
* Uploading the IG Publisher build output [directly](#upload) to a website
* Using the [IG Publisher `-go-publish`](#using-ig-publishers--go-publish-function) function to stage an IG publication, then upload it to a website.

Alternatively, the IG could be authored and published through [Firely](https://fire.ly)'s [Simplifier](https://simplifier.net) product.

Before briefly discussing each of these approaches, let's consider the goals of "publication" and when it is typically appropriate. Obviously, an author wants to publish their Implementation Guide to get it in front of a wider audience. But for what purpose? The author may be seeking feedback on an in-process guide, or may want an "official" release made available. Implementers want a fixed, identifiable version to base their implementation on; a version that aligns with what their exchange partners are using for their implementations. Although it is possible to publish, and keep, every version throughout the development process of the guide, most authors won't do this. Only key versions are published, often along side the single, most recent development (or continuous integration, CI) build. Once published, there is usually an expectation that an IG remain available to the readership for historical reference. Along with publication, most authors want the various FHIR registries to list their guide and to index the technical content, in the form of an NPM package.

### CI Auto-build

IG Publisher users who store their projects on [Github](http://github.com) can make use of the [IG auto-build](https://github.com/FHIR/auto-ig-builder) to "publish" their IG to the FHIR continuous integration site under `build.fhir.org`. This is a convenient way to allow others to see your implementation guide.

However, there are some drawbacks:

* Warnings at the top of each page of the IG that the document is "not an authorized publication," is the "continuous integration build," and "changes regularly."
* Canonical URLs are not permitted to be located under <http://build.fhir.org>
* Canonical URLs are not redirected to the appropriate resource or resource page (see below)
* Extremely limited support for multiple releases. Although different branches of an IG repo can be uploaded, there is no cross-referencing between them.
* No history page is generated, and any links to "directory of published versions" are broken
* No registration of the IG and IG package with FHIR tooling feeds
* Unstable availability (see below)

"Canonical redirect" is the resolution of the canonical URLs which identify the IG as well as IG profiles, CodeSystem, ValueSet, and so on, to the appropriate content. For example, the url [`http://example.com/ig/myig/StructureDefinition/MyPatient`](http://example.com/ig/myig/StructureDefinition/MyPatient) is expected to return an HTML descriptive page, or JSON- or XML-formatted StructureDefinition resource representation depending on the web request. This capability depends on both the IG being published at the location that the canonical URL references, and having appropriate website configuration or redirect scripts.

The fact that the CI build content changes any time an update is made to your repository makes using the auto-build unsuitable for long-term availability of milestone releases. Further, CI build content for your IG's non-main branch may be purged to save space; the main branch content has also occasionally been purged. The CI auto-build is not a good approach to long-term publishing.

On the other hand, the auto-build is a decent way to make your continuous integration builds, or other temporary IG versions, available for a limited time, such as sharing your in-progress work with your collaborators.

Note that the IG Publisher publication process described in this IG can reference auto-built IGs as part of the history of published versions, and in the registry feeds.

### Simple Upload to a Web Server or Github Pages {#upload}

To make your content available, another option is to upload the contents of the IG Publisher-generated `output` directory *directly* to a web server, or to copy it to [Github Pages](http://github.io). (Setting up automated builds and copying to Github Pages is covered in the [author's blog](http://www.argentixinfo.com/archives/156).)

This is essentially the same as using the auto-build, except that the output ends up on your website, rather than on [build.fhir.org](http://build.fhir.org/ig). And, this approach has many of the same issues as the CI auto-build.

* A "local development build" warning box on each page of the IG
* Canonical URLs are not redirected to the appropriate resource or resource page (although your web server configuration or scripts could theoretically address this)
* Extremely limited support for multiple releases. Although different releases of an IG could be uploaded, there is no cross-referencing between them.
* No history page is generated, and any links to "directory of published versions" are broken
* No registration of the IG and IG package with FHIR tooling feeds

Although uploading to your own web server won't be subject to unplanned removal of IG versions, it offers few other benefits compared to use of the auto-build.

The limitations described here around using your own website or GitHub pages are only applicable when doing a simple upload. This IG describes processes to address these issues. Also, the process described in this IG can reference continuous integration builds uploaded to your website as part of the history of published versions, and in the registry feeds.

It is worth noting that certain limitations, such as not being included in the registries, may be desirable in some cases, for example, to authors working on non-public guides.

### Publication using Simplifier

One of key and early decisions in authoring an IG is whether to use HL7's IG publisher or [Firely](https://fire.ly)'s [Simplifier](https://simplifier.net). Factors to consider between the two include:

* Commercial vs open source model, and associated factors including cost, license, support, stability
* Enforcement of "best practices," error checking and "correctness"
* Integration with resource authoring tools such as [Firely's Forge](https://fire.ly/products/forge/) or HL7's [FSH](http://hl7.org/fhir/uv/shorthand/) language compiler, [SUSHI](https://fshschool.org). (Both tools can be used with either publication mechanism)
* Command-line "compiler"-style interface vs. WYSIWYG
* Content management and versioning approach during authoring (including "release" management and Github integration)
* Control over look-and-feel, ADA compliance, and multi-language support
* Local vs. online; desktop control vs. SaaS.

However, for the purposes of this IG, the biggest difference is that a Simplifier license includes a website to publish the IG to. IG Publisher requires you to have a website to host your publication.

Publishing on your own website provides an additional level of control, and sense of ownership, that many prefer. Note though, that it is possible to download and self-host a Simplifier IG.

If you use the Simplifier development tooling, the publication considerations discussed in this document don't apply. As pointed out above, though, publication considerations should be only one part of the decision. Both the IG publisher and Simplifier continue to evolve, and the comparative features and differentiators will change.

### Using IG Publisher's `-go-publish` Function

The HL7 IG Publisher can be used to build an IG from source resources and narrative content. The output of an IG Publisher build may be uploaded to a website or made available through the CI auto-build, as described above. Using the IG Publisher's `-go-publish` functionality, it can also be used to prepare the generated content for publication, addressing several deficiencies in the basic output:

* The box at the top of each page is modified to indicate that this is a proper publication of the IG; it indicates which IG version you are viewing, what the current (most recent) version is, and includes a link to a history page with all published IG versions listed.
* The necessary redirect configuration or scripts are included so that canonical URLs in your IG will resolve correctly depending on whether the requesting client prefers HTML, JSON or XML.
* The publication directory is structured to support multiple IG publications.
* Updates to the publication feed files are prepared, so that the IG publication can be registered with HL7 tooling.

Once built using IG publisher, the IG can be uploaded to a website for wider availability.

Using this `-go-publish` functionality is the subject of this IG.

### Next Steps

The remainder of this implementation guide discusses the one-time process needed to [set up](./setup.html) an environment for publication, and the process needed to publish each release of [each publication](./publication.html).

As a pre-requisite for the processes described in this guide, you should have write access to a website that you control, and be familiar with the process for uploading content to it. This guide assumes your publication website aligns with the canonical URLs for your IGs. (Alignment of canonicals is not strictly necessary; however, this is the most common approach, and some functionality won't work unless the canonicals align.) You should also be familiar with basic use of the IG Publisher, use of the `git` source control tool, and have access to the [Github](http://github.com) website. Github access is needed to obtain or modify certain HL7 resources. The guide describes use of Github to manage your IG as well, however you may use other (or no) content management tools for your IG and publication.

[References](./references.html) to background material are included, as is a trivial profile of a [resource](./artifacts.html) which can be used to observe canonical resolution behaviour.

<div class="dragon" markdown=1>
The HL7 IG Publisher and publishing ecosystem (templates, terminology servers, registries, etc.) are under continual improvement. The content of this IG is believed to correctly describe the processes as it exists at the time of publication. If the process changes, or an inaccuracy is found, please provide let the IG author know. The sources listed on the [references](./references.html) page may be more up-to-date, however, they may also lag.
</div>
<br>
