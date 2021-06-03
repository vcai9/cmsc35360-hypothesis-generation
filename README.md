# cmsc35360-hypothesis-generation

## Approach 1
### Preprocessing
My dataset was the MeTeOR flat edge lists. I first disambiguated the CIDs and EntrezIDs by appending a "C" in front of any CIDs and an "E" in front of any EntrezIDs. This notation is used in the output CSV files. I then concatenated the six edge lists and used the Python library `networkx` to generate a graph of the entities. I removed any entities with less than 10 neighbors from consideration, which made the task more computationally feasible.  
### Method
For my first approach, I used collaborative filtering with SVD. Instead of using the confidence directly, I took its log and used that to get a more informative range of ratings. While the median confidence was just 2 and the average was about 20, the maximum confidence was 151,305; meanwhile, the range after taking the log was from 0 to 15. I used the `surprise` library's implementation of SVD. After training the model on all 9,009,864 edges in the dataset, I predicted on 804,700,000 non-edges (not including nodes with fewer than 10 neighbors) and maintained a heap of the 1,000 best hypotheses by their predicted rating. The results are in `svd.csv`.  
## Approach 2
I tried a number of approaches. First, I used the same library as above but with KNN as the algorithm; however, this ran for several hours but ended up crashing. I also tried several simple graph-based approaches. In one, I attempted to find the number of paths of different lengths between the nodes of a hypothesized pair, with more weight placed on paths of shorter length—predictably, this took far too long to compute on even small subsets of the dataset.  

I ended up using a simple metric, the Jaccard measure, to compute the relative number of neighbors in common, under the assumption that nodes with many mutual neighbors were more likely to be related themselves. However, simply using the number of mutual neighbors as our confidence would've biased our top hypotheses in favor of entities with many neighbors, so I had to take into account the number of total neighbors of the pair. Because I also had trouble getting this to run quickly, I ended up using just the gene-gene dataset. My results are in `jaccard.csv`. 

There are a few obvious issues with this approach. The first is that the confidence measure of the MeTeOR links isn't taken into account (ideally, we would probably want to weigh neighbors that have stronger links more). The second is that because I didn't trim any of the nodes from this dataset, the results are dominated by edges comprised of small-degree nodes. For instance, if two nodes both share one neighbor and have no other edges, then the Jaccard measure for that potential edge is 1. In the future, I'd first trim nodes with degree less than, perhaps, 10, and then retry this approach. 

## Potential Experiments
How to test these hypotheses in an automated laboratory would depend on which combination of genes, diseases, and chemicals we're dealing with. Many of these hypotheses, like gene-disease, might need to be tested in vivo or over long timescales and might not be suitable for an automated laboratory approach with human subjects. However, one might be able to autonomously run gene-disease experiments on nonhuman subjects by knocking out genes of interest with CRISPR or other tools to see the impact on disease, though there would likely still need to be significant human intervention. Others, like gene-gene (and to some extent, disease-disease), might only be apparent with many datapoints (given the sheer number of potential confounders), but could use automation to speed up the genomic sequencing and computation to speed up tests for pairwise interactions. With gene-chemical interactions (i.e. some chemical increases or decreases the expression of a gene), an automated setup could identify a potential interaction, expose human tissue to a given chemical, and measure the change in gene expression that results. Similarly, with disease-chemical interactions, we could run ex vivo experiments which expose tissue to chemicals and measure some relevant metric of success (e.g. shrinking of tumors, viral load, etc.).
