# Generating Procedural Materials from Text or Image Prompts
[Website](https://yiweihu.netlify.app/uploads/hu2023gen/project.html) | [Paper](https://graphics.cs.yale.edu/sites/default/files/generating_procedural_materials_from_text_or_image_prompts.pdf)

## Abstract
Node graphs are amazing, but so hard to create (require expertise). This paper aims to create a model to generate node graphs, either conditionally from image or text prompts, or unconditionally. Work is inspired by MatFormer 
(which was only unconditional generation). They also created a material graph dataset (which, in SIGGRAPH, they said won't be published to the public, at least for now). The results are impressive, better than novel
CLIP-based datasearch pipelines.
(Note: if considering only the image input, this would be an inverse rendering problem!)

## Related Work
- MatFormer (transformer-based generative model for material graphs). What's better: MatFormer relies on a generic graph structure. This work does not and can generate novel graph structures.
- CLIP (joint image/text embedding)
- Material acquisition & generation: other works attempt to correctly estimate material properties that outputs texels directly (I'm not too sure how this works), but this work attempts to instead generate the node graphs,
  which in turn can be used to generate material parameter maps (to future self: if wondering what a material parameter map is, read the [Dataset](#dataset) section)

## Method (Graph generation)
- Goal: Model conditional distribution $p(g|y)$ of graphs $g$ given input $y$.
- Encode the graph into 3 sequences: node sequence, edge sequence, parameter sequence (this can be obtained by linearizing the graph - note: this just means traversing the graph in a certain order.
  Reason for linearizing the graph: maybe so that it is easier to optimize?). To read more about the structure of the node graphs, read section 3.2
- Training the conditional generative model: to improve on MatFormer, they add $y$ as the condition. The order of node graph generation is as followed: node generation, conditioned on $y$ => edge generation, conditioned on nodes, and $y$ => parameter generation, conditioned on nodes, edges, and $y$.
  To see the full model graph, look at Figure 2.
- Conditioning on text prompts: a bit different from images => after text is transformed into CLIP text embeddings, apply a prior (DALL-E2's approach) to transform CLIP text embeddings into CLIP texture embeddings.
- Conditioning on image prompts: images can be used to extract low-level information that CLIP texture embeddings lack. They therefore add VGG feature statistics to capture this information - unfortunately this cannot
  be applied to text prompts.
- Actual generation: generating nodes, edges & parameters: I'm confused by how they condition next tokens on previously generated tokens. Why do this? The tokens are not always related (they are linearized), this doesn't make sense

## Dataset
- Dataset is complicated and a material graph can generate versions of material parameter maps that are, while useful for artists (for different workflows), not useful for them. They therefore choose to focus on 1 workflow,
  the PBR (physically based rendering) workflow where a graph generates albedo, normal, roughness and metallic maps)
