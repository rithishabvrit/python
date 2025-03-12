import argparse
import csv
from Bio import Entrez, Medline

def fetch_papers(query):
    Entrez.email = 'rithisha.maloth.com'
    handle = Entrez.esearch(db='pubmed', term=query, retmax=100)
    record = Entrez.read(handle)
    id_list = record['IdList']

    papers = []
    for id in id_list:
        handle = Entrez.efetch(db='pubmed', id=id, rettype='medline', retmode='text')
        record = Medline.read(handle)
        paper = {
            'PubmedID': id,
            'Title': record.get('TI', ''),
            'Publication Date': record.get('DP', ''),
            'Non-academic Author(s)': '',
            'Company Affiliation(s)': '',
            'Corresponding Author Email': ''
        }
        papers.append(paper)

    return papers

def save_to_csv(papers, filename):
    with open(filename, 'w', newline='') as csvfile:
        fieldnames = ['PubmedID', 'Title', 'Publication Date', 'Non-academic Author(s)', 'Company Affiliation(s)', 'Corresponding Author Email']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        for paper in papers:
            writer.writerow(paper)

def main():
    parser = argparse.ArgumentParser(description='Fetch papers from PubMed')
    parser.add_argument('query', help='PubMed query')
    parser.add_argument('-o', '--output', help='Output filename')
    parser.add_argument('-d', '--debug', action='store_true', help='Print debug information')
    args = parser.parse_args()

    if args.debug:
        print('Debug mode enabled')

    papers = fetch_papers(args.query)

    if args.output:
        save_to_csv(papers, args.output)
    else:
        for paper in papers:
            print(paper)

if __name__ == '__main__':
    main()
